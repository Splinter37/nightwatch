### Changes:

- Static user id = 0 (make loop on KeacloakAuth)
        
        in: src/Sensors/CacheEventSensor.php

        - 'user' => $this->executionState->user->id(),
        + 'user' => '0',
- Skip NightwatchServiceProvider for Livewire update path

        in: src/NightwatchServiceProvider.php
        
        if($this->isRequest && config('nightwatch.filtering.ignore_livewire_update') && str_contains(request()->path(), 'liwevire/update')) {
            \Log::emergency('Request skipped in Provider', ['path' => $request->path()]);
            return;
        }


- Add excluded Requests, Commands, ScheduledTasks and JobAttempts 

        in: src/Concerns/CapturesState.php 
        
        public function jobAttempt(JobProcessed|JobReleasedAfterException|JobFailed $event): void
        {
            try {
                $jobData = json_decode($event->job->getRawBody(), true, 512, JSON_THROW_ON_ERROR);
    
                $skip = isset($jobData['displayName'])
                    && in_array($jobData['displayName'], config('nightwatch.exclude.jobs') ?? [], true);
    
                if($skip) {
                    return;
                }
            } catch (Throwable $e) {
                \Log::error('Nightwatch jobAttemp catch', ['path' => $e->getMessage()]);
            }
        ...

  

        public function command(InputInterface $input, int $status): void
        {
            try {
                $skip = in_array($this->executionState->name, config('nightwatch.exclude.commands') ?? [], true);
    
                if($skip) {
                    return;
                }
            } catch (Throwable $e) {
                \Log::error('Nightwatch command catch', ['path' => $e->getMessage()]);
            }
        ...

        public function scheduledTask(ScheduledTaskFinished|ScheduledTaskSkipped|ScheduledTaskFailed $event): void
        {
            try {
                $skip = collect(config('nightwatch.exclude.scheduled_tasks', []))
                    ->filter(fn (string $item): bool => str_contains($item, $this->executionState->name))
                    ->count() > 0;

                if($skip) {
                    return;
                }
            } catch (Throwable $e) {
                \Log::error('Nightwatch scheduled task catch', ['path' => $e->getMessage()]);
            }
        ...
