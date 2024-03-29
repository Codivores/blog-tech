---
title: "Laravel - Backup your filesystem disks"
seoTitle: "Laravel Backup: How to back up External Filesystem Disks"
seoDescription: "Learn how to efficiently back up your Laravel application's external disks using the powerful Laravel Backup package with a step-by-step guide"
datePublished: Thu Jun 08 2023 08:48:31 GMT+0000 (Coordinated Universal Time)
cuid: climwbk3p000n08mg4p571qq6
slug: laravel-backup-external-filesystem-disks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686036807580/7a66a772-6be2-4530-a877-fd1b757eada0.png
tags: cloud, laravel, php, backup, spatie

---

You are probably already familiar with the work of Spatie and its contributors in the Laravel and Open Source ecosystem.

For many years, we have been using their Backup package ([https://spatie.be/docs/laravel-backup](https://spatie.be/docs/laravel-backup)) to back up our applications (databases and assets) on cloud provider's object storage like S3 or OpenStack Swift.

As our projects started utilizing more and more external disks to store assets and files generated by the application, such as PDF files, we encountered a limitation with the package — it only supports the backup of local files.

In this article, we will explore how to extend the package to back up external Laravel disks (S3 compatible, OpenStack Swift, ...).

# Deep dive into the Package code

Let's take a closer look at the code of the package. Upon inspection, we can understand why it only handles local files:

* it uses the `Symfony Finder` to locate the files to back up
    
* it creates a `temporary local folder` to create a `Zip file` before sending it to the configured destination disks (after the backup process is complete, the temporary folder is deleted)
    

Making the Symfony Finder work with other filesystem types would require significant effort, and downloading the content of external disks to create a Zip file locally, and then sending it to other disks, is not an ideal solution...

However, there is good news. The package dispatches several events, such as when the Zip file is created, when the backup fails or succeeds, ...

# The Approach

Based on these observations, we can devise a strategy to back up our external disks:

* Instead of creating a Zip file, we will perform a direct copy of the files to the destination disks
    
* We will listen to the following events to trigger our custom processes:
    
    * **BackupWasSuccessful**: to launch the copy operation
        
    * **CleanupWasSuccessful**: to clean up old backups
        

# Implementation

## Configuration

First, let's add a new entry in the package's configuration file to specify the disks we want to back up.

**/config/backup.php**

```php
<?php

return [
    'backup' => [
        'source' => [
            // ...
            'disks' => [
                'assets',
                'customers',
            ],
            // ...
        ],
];
```

## The logic

We will create a Subscriber to handle the events and implement our custom processes.

### BackupWasSuccessful event

The `BackupWasSuccessful` event is the most suitable event to perform our custom copy operation. It is dispatched when the standard backup job has finished its work.

By accessing the `backupDestination` instance provided by the event, we can obtain the path of the Zip file on the destination disk.

Here's how we proceed:

* we compute the backup path. To simplify backup and cleanup,
    
    * the directory that will be created will have the same name as the Zip file
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686213827566/3da0eeaa-b630-4c07-8774-c1ae57b4b64f.png align="right")
    
    * the files / directories will be copied into a subdirectory that will have the same name as the external disk (here we back up two disks: assets and customers)
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686213770139/c94cb667-755e-4ba4-8b94-320a25d2b920.png align="right")

* we retrieve all the external disks we want to back up from the package's configuration file, and for each disk:
    
    * we retrieve all the files, including subdirectories.
        
    * we copy each file to the backup disk configured in the package.
        

### CleanupWasSuccessful event

The `CleanupWasSuccessful` event is the most suitable event to perform our custom cleanup. It is dispatched when the standard cleanup job has finished its work.

Similar to the previous event, we can access the `backupDestination` instance provided by the event.

Here's how we proceed:

* we retrieve all the directories on the destination disk. These directories are expected to contain only the custom backups  
    ❗All directories not related to backups in the backup directory will be deleted
    
* we retrieve all of the existing Zip backups from the `backupDestination` (we add `fresh()` method call because the Job removes the last backup from the collection, and as it's cached, we need to retrieve the updated list)
    
* we compute the directories to delete by identifying the directories that do not have a corresponding Zip file.
    
* we delete the directories that need to be cleaned up
    

## Subscriber code

**/app/Listeners/BackupEventSubscriber.php**

```php
<?php

namespace App\Listeners;

use Illuminate\Events\Dispatcher;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;
use Spatie\Backup\Events\BackupWasSuccessful;
use Spatie\Backup\Events\CleanupWasSuccessful;
use Symfony\Component\Console\Output\ConsoleOutput;

class BackupEventSubscriber
{
    public function subscribe(Dispatcher $events): array
    {
        return [
            BackupWasSuccessful::class => 'handleBackup',
            CleanupWasSuccessful::class => 'handleCleanup',
        ];
    }

    public function handleBackup(BackupWasSuccessful $event): void
    {
        $backupDestination = $event->backupDestination;
        $backupPath = Str::replaceLast('.zip', '', $backupDestination->newestBackup()->path());

        $output = new ConsoleOutput();

        $sources = config('backup.backup.source.disks') ?? [];

        foreach ($sources as $disk) {
            $output->writeln('<info>Determining files to backup for disk ' . $disk . '...</info>');
            $destinationPath = $backupPath . '/' . $disk . '/';

            $files = Storage::disk($disk)->allFiles();
            $output->writeln('<info>Copying disk ' . $disk . ' to disk named backup: ' . count($files) . ' files</info>');

            foreach ($files as $file) {
                Storage::disk($backupDestination->diskName())->put(
                    $destinationPath . $file,
                    Storage::disk($disk)->get($file)
                );
            }
        }
    }

    public function handleCleanup(CleanupWasSuccessful $event): void
    {
        $backupDestination = $event->backupDestination;

        $output = new ConsoleOutput();
        $output->writeln('<info>Cleaning disks backups of ' . $backupDestination->backupName() . ' on disk ' . $backupDestination->diskName() . '...</info>');

        $directories = Storage::disk($backupDestination->diskName())->directories($backupDestination->backupName());

        if (count($directories) > 0) {
            $backups = $backupDestination
                ->fresh()
                ->backups()
                ->map(function ($backup) {
                    return Str::replaceLast('.zip', '', $backup->path());
                })
                ->toArray();

            $directoriesToDelete = array_filter($directories, function ($directory) use ($backups) {
                return !in_array($directory, $backups);
            });

            if (count($directoriesToDelete) > 0) {
                foreach ($directoriesToDelete as $directory) {
                    Storage::disk($backupDestination->diskName())->deleteDirectory($directory);
                }
            }
        }
    }
}
```

## Register the Subscriber

The final step is to register the Subscriber in the Laravel `EventServiceProvider`

**/app/Providers/EventServiceProvider.php**

```php
<?php

namespace App\Providers;

use App\Listeners\BackupEventSubscriber;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * The subscriber classes to register.
     *
     * @var array
     */
    protected $subscribe = [
        BackupEventSubscriber::class,
    ];
}
```

---

%[https://media.giphy.com/media/26Ffd0AF5J2kJgWhG/giphy.gif]