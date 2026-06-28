# @codenameryuu/adonis-scheduler

This addon adds the functionality to schedule tasks in Adonis JS 7.

## Requirement

* Adonis Js 7

## Installation

* Install the package 

```bash
yarn add @codenameryuu/adonis-scheduler
```

* Add the package to the `adonisrc.ts` file

```bash
node ace add @codenameryuu/adonis-scheduler
```

## Running The Scheduler

```sh
node ace scheduler:run
# or
node ace scheduler:work

# automatically restart the scheduler when files are modified during development mode
node ace scheduler:run --watch

node ace scheduler:run --tag <tag_name> # Run all schedules with the specified tag
```

## Defining Schedules

```ts
// start/scheduler.ts

import scheduler from '@codenameryuu/adonis-scheduler/services/main'

import PurgeUsers from '../commands/purge_users'

scheduler.command('inspire').everyFiveSeconds()
scheduler.command(PurgeUsers, ['30 days']).everyFiveSeconds().withoutOverlapping()

scheduler.withoutOverlapping(
  () => {
    scheduler.command('inspire').everySecond()
    scheduler.command(PurgeUsers, ['30 days']).everyFiveSeconds()
  },
  { expiresAt: 30_000 }
)

scheduler.withTag(
  () => {
    scheduler.command('inspire').everySecond()
  },
  'my-tag'
)

scheduler
  .call(() => {
    console.log('Pruge DB!')
  })
  .weekly()

scheduler
  .call(() => {
    console.log('Pruge DB!')
  })
  .tag('my-tag')
  .weekly()
```

or define schedule directly on command

```ts
import { BaseCommand, args } from '@adonisjs/core/ace'
import { schedule } from '@codenameryuu/adonis-scheduler'

@schedule("* * * * *", ["30 days"])
@schedule((s) => s.everyFiveSeconds().immediate(), ["7 days"])
@schedule((s) => s.everyMinute(), ["42 days"])
export default class PurgeUsers extends BaseCommand {
  static commandName = 'purge:users'
  static description = ''

  static options: CommandOptions = {}

  @args.string()
  declare olderThan: string

  async run() {
    //
  }
}
```

## Schedule Frequency Options

| Method                           | Description                                             |
| -------------------------------- | ------------------------------------------------------- |
| `.cron('* * * * *');` | Run the task on a custom cron schedule                  |
| `.everyMinute();` | Run the task every minute                               |
| `.everyTwoMinutes();` | Run the task every two minutes                          |
| `.everyThreeMinutes();` | Run the task every three minutes                        |
| `.everyFourMinutes();` | Run the task every four minutes                         |
| `.everyFiveMinutes();` | Run the task every five minutes                         |
| `.everyTenMinutes();` | Run the task every ten minutes                          |
| `.everyFifteenMinutes();` | Run the task every fifteen minutes                      |
| `.everyThirtyMinutes();` | Run the task every thirty minutes                       |
| `.hourly();` | Run the task every hour                                 |
| `.hourlyAt(17);` | Run the task every hour at 17 minutes past the hour.    |
| `.everyTwoHours();` | Run the task every two hours                            |
| `.everyThreeHours();` | Run the task every three hours                          |
| `.everyFourHours();` | Run the task every four hours                           |
| `.everyFiveHours();` | Run the task every five hours                           |
| `.everySixHours();` | Run the task every six hours                            |
| `.daily();` | Run the task every day at midnight                      |
| `.dailyAt('13:00');` | Run the task every day at 13:00.                        |
| `.twiceDaily(1, 13);` | Run the task daily at 1:00 & 13:00.                     |
| `.twiceDailyAt(1, 13, 15);` | Run the task daily at 1:15 & 13:15.                     |
| `.weekly();` | Run the task every Sunday at 00:00                      |
| `.weeklyOn(1, '8:00');` | Run the task every week on Monday at 8:00.              |
| `.monthly();` | Run the task on the first day of every month at 00:00   |
| `.monthlyOn(4, '15:00');` | Run the task every month on the 4th at 15:00.           |
| `.twiceMonthly(1, 16, '13:00');` | Run the task monthly on the 1st and 16th at 13:00.      |
| `.lastDayOfMonth('15:00');` | Run the task on the last day of the month at 15:00.     |
| `.quarterly();` | Run the task on the first day of every quarter at 00:00.|
| `.quarterlyOn(4, '14:00');` | Run the task every quarter on the 4th at 14:00.         |
| `.yearly();` | Run the task on the first day of every year at 00:00.   |
| `.yearlyOn(6, 1, '17:00');` | Run the task every year on June 1st at 17:00.           |
| `.timezone('America/New_York');` | Set the timezone for the task.                          |
| `.immediate();` | Run the task on startup                                 |
| `.withoutOverlapping();` | Run the task without overlapping                        |

## Alternative Ways to Run the Scheduler

Besides using `node ace scheduler:run` , you can also manually initialize and control the scheduler worker in your code:

```ts
import { Worker } from '@codenameryuu/adonis-scheduler'
import app from '@adonisjs/core/services/app'

const worker = new Worker(app)

app.terminating(async () => {
  await worker.stop()
})

await worker.start()
```

Note: make sure to modify `adonisrc.ts` and include the environment where the worker is running.

```ts
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
 providers: [
   {
      file: () => import('@codenameryuu/adonis-scheduler/scheduler_provider'),
      environment: ['console', 'web'], // <---
   },

   // or enable for all environments
   () => import('@codenameryuu/adonis-scheduler/scheduler_provider'),
})
```
