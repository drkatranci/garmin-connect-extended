# garmin-connect-extended

> Extended version with enhanced golf shot data support

## About This Fork

This is an extended version of the original `garmin-connect` library with additional golf functionality:

-   ✅ **Enhanced Golf Data**: Access detailed shot-by-shot golf data including club selection, distances, and accuracy metrics
-   ✅ **Shot Analysis**: Retrieve individual shot information for specific holes or entire scorecards
-   ✅ **Extended API Coverage**: Additional Garmin Connect golf endpoints for comprehensive golf analytics

## Extended Features

### Golf Shot Data

-   `getGolfScorecardHole()` - Retrieve detailed shot data for specific holes in a golf scorecard
-   Access to club usage, shot distances, accuracy metrics, and shot coordinates
-   Support for filtering by specific hole numbers

---

## v1.6.0 refactor

TODO:

-   [x] New HttpClient class
-   [x] Login and get user token
-   [x] Garmin URLs works with `garmin.cn` and `garmin.com`
-   [x] Auto refresh Ouath2 token
-   [x] Oauth1,Oauth2 token import and export.
-   [x] Download Activity, countActivities, getActivities, getActivity, getUserProfile, getUserSettings
-   [x] Upload Activity, delete Activity
-   [ ] Implementation of other methods, such as Badge,Workout,Gear etc
-   [ ] Handle MFA
-   [x] Handle Account locked
-   [ ] Unit test
-   [ ] Listeners

If something is not working, please check [https://connect.garmin.com/status/](https://connect.garmin.com/status/) first.

Currently, most of previous features are working, but some of Rest API are not added, such as `Gear`,`Workout`,`Badge` etc. So if you need these features, please add a PR.

All of above work inspired by [https://github.com/matin/garth](https://github.com/matin/garth). Many thanks.

---

A powerful JavaScript library for connecting to Garmin Connect for sending and receiving health and workout data. This **extended version** includes enhanced golf functionality for detailed shot analysis and scorecard data. It comes with some predefined methods to get and set different kinds of data for your Garmin account, but also have the possibility to make [custom requests](#custom-requests) `GET`, `POST` and `PUT` are currently supported. This makes it easy to implement whatever may be missing to suite your needs.

## Golf Enhancement Features

This extended version adds comprehensive golf shot tracking capabilities:

-   **Detailed Shot Data**: Access individual shot information including club selection, distances, and accuracy
-   **Hole-by-Hole Analysis**: Retrieve shot data for specific holes or entire rounds
-   **Enhanced Golf APIs**: Additional endpoints for comprehensive golf performance analysis

## Prerequisites

This library will require you to add a configuration file to your project root called `garmin.config.json` containing your username and password for the Garmin Connect service.

```json
{
    "username": "my.email@example.com",
    "password": "MySecretPassword"
}
```

## How to install

### From GitHub (Recommended for Extended Features)

```shell
$ npm install github:drkatranci/garmin-connect-extended
```

### Alternative Installation Methods

```shell
# Direct GitHub install
$ npm install drkatranci/garmin-connect-extended

# Or add to package.json
{
  "dependencies": {
    "garmin-connect": "github:drkatranci/garmin-connect-extended"
  }
}
```

### Original Package (Without Golf Extensions)

```shell
$ npm install garmin-connect
```

## How to use

```js
const { GarminConnect } = require('garmin-connect');
// Create a new Garmin Connect Client
const GCClient = new GarminConnect({
    username: 'my.email@example.com',
    password: 'MySecretPassword'
});
// Uses credentials from garmin.config.json or uses supplied params
await GCClient.login();
const userProfile = await GCClient.getUserProfile();
```

Now you can check `userProfile.userName` to verify that your login was successful.

## Golf Shot Data Example (Extended Feature)

```js
const { GarminConnect } = require('garmin-connect');

const GCClient = new GarminConnect({
    username: 'my.email@example.com',
    password: 'MySecretPassword'
});

await GCClient.login();

// Get golf summary to find scorecard IDs
const golfSummary = await GCClient.getGolfSummary();
console.log('Available scorecards:', golfSummary);

// Get detailed shot data for a specific scorecard
const scorecardId = '12345'; // Replace with actual scorecard ID
const shotData = await GCClient.getGolfScorecardHole(scorecardId);
console.log('All shot data:', shotData);

// Get shot data for specific holes only
const specificHoles = await GCClient.getGolfScorecardHole(scorecardId, '1,2,3');
console.log('Holes 1-3 shot data:', specificHoles);

// Example: Analyze driver usage
shotData.holes?.forEach((hole) => {
    hole.shots?.forEach((shot) => {
        if (shot.club === 'Driver') {
            console.log(
                `Hole ${hole.holeNumber}: Driver shot ${shot.distance} yards`
            );
        }
    });
});
```

## Reusing your session(since v1.6.0)

### Save token to file and reuse it.

```js
GCClient.saveTokenToFile('/path/to/save/tokens');
```

Result:

```bash
$ ls /path/to/save/tokens
oauth1_token.json oauth2_token.json
```

Reuse token:

```js
GCClient.loadTokenByFile('/path/to/save/tokens');
```

### Or just save your token to db or other storage.

```js
const oauth1 = GCClient.client.oauth1Token;
const oauth2 = GCClient.client.oauth2Token;
// save to db or other storage
...
```

Reuse token:

```js
GCClient.loadToken(oauth1, oauth2);
```

## Reusing your session(deprecated)

This is an experimental feature and might not yet provide full stability.

After a successful login the `sessionJson` getter and setter can be used to export and restore your session.

```js
// Exporting the session
const session = GCClient.sessionJson;

// Use this instead of GCClient.login() to restore the session
// This will throw an error if the stored session cannot be reused
GCClient.restore(session);
```

The exported session should be serializable and can be stored as a JSON string.

A stored session can only be reused once and will need to be stored after each request. This can be done by attaching some storage to the `sessionChange` event.

```js
GCClient.onSessionChange((session) => {
    /*
        Your choice of storage here
        node-persist will probably work in most cases 
     */
});
```

### Login fallback

To make sure to use a stored session if possible, but fallback to regular login, one can use the `restoreOrLogin` method.
The arguments `username` and `password` are both optional and the regular `.login()` will be
called if session restore fails.

```js
await GCClient.restoreOrLogin(session, username, password);
```

## Events

-   `sessionChange` will trigger on a change in the current `sessionJson`

To attach a listener to an event, use the `.on()` method.

```js
GCClient.on('sessionChange', (session) => console.log(session));
```

There's currently no way of removing listeners.

## Reading data

### User info is not implemented yet. // TODO: Implement this function

Receive basic user information

```js
GCClient.getUserInfo();
```

### Social Profile is not implemented yet. // TODO: Implement this function

Receive social user information

```js
GCClient.getSocialProfile();
```

### Social Connections is not implemented yet. // TODO: Implement this function

Get a list of all social connections

```js
GCClient.getSocialConnections();
```

### Device info is not implemented yet. // TODO: Implement this function

Get a list of all registered devices including model numbers and firmware versions.

```js
GCClient.getDeviceInfo();
```

### `getActivities(start: number, limit: number, activityType?: ActivityType, subActivityType?: ActivitySubType): Promise<IActivity[]>`

Retrieves a list of activities based on specified parameters.

#### Parameters:

-   `start` (number, optonal): Index to start fetching activities.
-   `limit` (number, optonal): Number of activities to retrieve.
-   `activityType` (ActivityType, optional): Type of activity (if specified, start must be null).
-   `subActivityType` (ActivitySubType, optional): Subtype of activity (if specified, start must be null).

#### Returns:

-   `Promise<IActivity[]>`: A Promise that resolves to an array of activities.

#### Example:

```js
const activities = await GCClient.getActivities(
    0,
    10,
    ActivityType.Running,
    ActivitySubType.Outdoor
);
```

### `getActivity(activity: { activityId: GCActivityId }): Promise<IActivity>`

Retrieves details for a specific activity based on the provided `activityId`.

#### Parameters:

-   `activity` (object): An object containing the `activityId` property.

    -   `activityId` (GCActivityId): Identifier for the desired activity.

#### Returns:

-   `Promise<IActivity>`: A Promise that resolves to the details of the specified activity.

#### Example:

```js
const activityDetails = await GCClient.getActivity({
    activityId: 'exampleActivityId'
});
```

### News Feed is not implemented yet. // TODO: Implement this function

To get a list of activities in your news feed, use the `getNewsFeed` method. This function takes two arguments, _start_ and _limit_, which is used for pagination. Both are optional and will default to whatever Garmin Connect is using. To be sure to get all activities, use this correctly.

```js
// Get the news feed with a default length with most recent activities
GCClient.getNewsFeed();
// Get activities in feed, 10 through 15. (start 10, limit 5)
GCClient.getNewsFeed(10, 5);
```

### Download original activity data

Use the activityId to download the original activity data. Usually this is supplied as a .zip file.

```js
const [activity] = await GCClient.getActivities(0, 1);
// Directory path is optional and defaults to the current working directory.
// Downloads filename will be supplied by Garmin.
GCClient.downloadOriginalActivityData(activity, './some/path/that/exists');
```

### Upload activity file

Uploads an activity file as a new Activity. The file can be a `gpx`, `tcx`, or `fit` file. If the activity already exists, the result will have a status code of 409.
Upload fixed in 1.4.4, Garmin changed the upload api, the response `detailedImportResult` doesn't contain the new activityId.

```js
const upload = await GCClient.uploadActivity('./some/path/to/file.fit');
// not working
const activityId = upload.detailedImportResult.successes[0].internalId;
const uploadId = upload.detailedImportResult.uploadId;
```

### Upload activity image

Uploads an image to activity

```js
const [latestActivty] = await GCClient.getActivities(0, 1);

const upload = await GCClient.uploadImage(
    latestActivty,
    './some/path/to/file.jpg'
);
```

### Delete activity image

Delete an image from activity

```js
const [activity] = await GCClient.getActivities(0, 1);
const activityDetails = await GCClient.getActivityDetails(activity.activityId);

await GCClient.deleteImage(
    activity,
    activityDetails.metadataDTO.activityImages[0].imageId
);
```

### `getSteps(date?: Date): Promise<number>`

Retrieves the total steps for a given date.

#### Parameters:

-   `date` (Date, optional): Date of the steps information requested; defaults to today if no date is supplied.

#### Returns:

-   `Promise<number>`: A Promise that resolves to the total steps for the specified date.

#### Example:

```js
const totalSteps = await GCClient.getSteps(new Date('2020-03-24'));
```

### `getSleepData(date: string): Promise<SleepData>`

Retrieves all sleep data for a given date

#### Parameters:

-   `date` (Date, optional): Date of information requested, this will default to today if no date is supplied

#### Returns:

-   `Promise<SleepData>`: A Promise that resolves to an object containing detailed sleep information.

    -   `dailySleepDTO` (object): Information about the user's daily sleep.
        -   `id` (number): The unique identifier of the sleep record.
        -   `userProfilePK` (number): The user's profile identifier.
        -   `calendarDate` (string): The date of the sleep record.
        -   ...
    -   `sleepMovement` (array): An array of sleep movement data.
    -   `remSleepData` (boolean): Indicates whether REM sleep data is available.
    -   `sleepLevels` (array): An array of sleep levels data.
    -   `restlessMomentsCount` (number): Count of restless moments during sleep.
    -   ...

#### Example:

```js
const detailedSleep = await GCClient.getSleepDuration(new Date('2020-03-24'));
```

### `getSleepDuration(date: string): Promise<{hours: number, minutes: number}`

Retrieves hours and minutes slept for a given date

#### Parameters:

-   `date` (Date, optional): Date of information requested, this will default to today if no date is supplied

#### Returns:

-   `Promise<{hours: string, minutes: string }>`: A Promise that resolves to an object containing information about the sleep duration

    -   `hours` (string): Number of hours
    -   `minutes` (string): Number of minutes

#### Example:

```js
const detailedSleep = await GCClient.getSleepDuration(new Date('2020-03-24'));
```

### `getDailyWeightData(date?: Date): Promise<number>`

Retrieves the daily weight and converts it from grams to pounds.

#### Parameters:

-   `date` (Date, optional): Date of information requested. Defaults to the current date.

#### Returns:

-   `Promise<number>`: A Promise that resolves to the daily weight converted from grams to pounds.

#### Throws:

-   `Error`: If valid daily weight data cannot be found for the specified date.

#### Example:

```js
const weightData = await GCClient.getDailyWeightData(new Date('2023-12-25'));
```

### `getDailyWeightInPounds(date?: Date): Promise<number>`

Retrieves the daily weight in pounds for a given date.

#### Parameters:

-   `date` (Date, optional): Date of information requested; defaults to today if no date is supplied.

#### Returns:

-   `Promise<number>`: A Promise that resolves to the daily weight in pounds.

#### Example:

```js
const weightInPounds = await GCClient.getDailyWeightInPounds(
    new Date('2020-03-24')
);
```

## `getDailyHydration(date?: Date): Promise<number>`

Retrieves the daily hydration data and converts it from milliliters to ounces.

### Parameters:

-   `date` (Date, optional): Date of the requested information. Defaults to the current date.

### Returns:

-   `Promise<number>`: A Promise that resolves to the daily hydration data converted from milliliters to ounces.

### Throws:

-   `Error`: If valid daily hydration data cannot be found for the specified date or if the response is invalid.

### Example:

```js
const hydrationInOunces = await GCClient.getDailyHydration(
    new Date('2023-12-25')
);
```

### `getGolfSummary(): Promise<GolfSummary>`

Retrieves a summary of golf scorecard data.

#### Returns:

-   `Promise<GolfSummary>`: A Promise that resolves to the golf scorecard summary.

#### Example:

```js
const golfSummary = await GCClient.getGolfSummary();
```

### `getGolfScorecard(scorecardId: number): Promise<GolfScorecard>`

Retrieves golf scorecard data for a specific scorecard.

#### Parameters:

-   `scorecardId` (number): Identifier for the desired golf scorecard.

#### Returns:

-   `Promise<GolfScorecard>`: A Promise that resolves to the golf scorecard data.

#### Example:

```js
const scorecardId = 123; // Replace with the desired scorecard ID
const golfScorecard = await GCClient.getGolfScorecard(scorecardId);
```

### `getGolfScorecardHole(scorecardId: string, holeNumbers?: string): Promise<any>`

Retrieves detailed golf shot data for specific holes in a scorecard. This method provides access to individual shot information including club used, distance, accuracy, and other shot metrics.

#### Parameters:

-   `scorecardId` (string): Identifier for the desired golf scorecard.
-   `holeNumbers` (string, optional): Comma-separated list of hole numbers to retrieve (e.g., "1,2,3"). If not provided, returns data for all holes.

#### Returns:

-   `Promise<any>`: A Promise that resolves to the golf shot data containing detailed information about each shot on the specified holes.

#### Example:

```js
// Get shot data for all holes in a scorecard
const allShotData = await GCClient.getGolfScorecardHole('12345');

// Get shot data for specific holes
const specificHoles = await GCClient.getGolfScorecardHole('12345', '1,2,3');

// Example response structure:
// {
//   holes: [
//     {
//       holeNumber: 1,
//       shots: [
//         {
//           shotNumber: 1,
//           club: "Driver",
//           distance: 280,
//           accuracy: "Fairway",
//           coordinates: { lat: 40.123, lng: -74.456 }
//         }
//       ]
//     }
//   ]
// }
```

### `getHeartRate(date?: Date): Promise<HeartRate>`

Retrieves daily heart rate data for a given date.

#### Parameters:

-   `date` (Date, optional): Date of the heart rate data requested; defaults to today if no date is supplied.

#### Returns:

-   `Promise<HeartRate>`: A Promise that resolves to the daily heart rate data.

#### Example:

```js
const heartRateData = await GCClient.getHeartRate(new Date('2020-03-24'));
```

## Modifying data

### Update activity is not implemented yet. // TODO: Implement this function

```js
const activities = await GCClient.getActivities(0, 1);
const activity = activities[0];
activity['activityName'] = 'The Updated Name';
await GCClient.updateActivity(activity);
```

### Delete an activity

Deletes an activty.

```js
const activities = await GCClient.getActivities(0, 1);
const activity = activities[0];
await GCClient.deleteActivity(activity);
```

### `updateHydrationLogOunces(date?: Date, valueInOz: number): Promise<WaterIntake>`

Adds a hydration log entry in ounces for a given date.

#### Parameters:

-   `date` (Date, optional): Date of the log entry; defaults to today if no date is supplied.
-   `valueInOz` (number): Amount of water intake in ounces. Accepts negative number.

#### Returns:

-   `Promise<WaterIntake>`: A Promise that resolves to the hydration log entry.

#### Example:

```js
const hydrationLogEntry = await GCClient.addHydrationLogOunces(
    new Date('2020-03-24'),
    16
);
```

### `updateWeight(date = new Date(), lbs: number, timezone: string): Promise<UpdateWeight>`

Updates weight information

#### Parameters:

-   `date` (optional): Date object representing the weight entry date. Defaults to the current date if not provided.
-   `lbs` (number): Weight value in pounds.
-   `timezone` (string): String representing the timezone for the weight entry.

#### Returns:

-   `Promise<UpdateWeight>`: A Promise that resolves to the result of the weight update.

#### Example:

```js
await GCClient.updateWeight(undefined, 202.9, 'America/Los_Angeles');
```

### Add workout

To add a custom workout, use the `addWorkout` or more specifically `addRunningWorkout`.

```js
GCClient.addRunningWorkout('My 5k run', 5000, 'Some description');
```

Will add a running workout of 5km called 'My 5k run' and return a JSON object representing the saved workout.

### Schedule workout

To add a workout to your calendar, first find your workout and then add it to a specific date.

```js
const workouts = await GCClient.getWorkouts();
const id = workouts[0].workoutId;
GCClient.scheduleWorkout({ workoutId: id }, new Date('2020-03-24'));
```

This will add the workout to a specific date in your calendar and make it show up automatically if you're using any of the Garmin watches.

### Delete workout

Deleting a workout is very similar to [scheduling](#schedule-workout) one.

```js
const workouts = await GCClient.getWorkouts();
const id = workouts[0].workoutId;
GCClient.deleteWorkout({ workoutId: id });
```

## Custom requests

This library will handle custom requests to your active Garmin Connect session. There are a lot of different url's that is used, which means that this library probably wont cover them all. By using the network analyze tool you can find url's that are used by Garmin Connect to fetch data.

Let's assume I found a `GET` requests to the following url:

```
https://connect.garmin.com/modern/proxy/wellness-service/wellness/dailyHeartRate/22f5f84c-de9d-4ad6-97f2-201097b3b983?date=2020-03-24
```

The request can be sent using `GCClient` by running

```js
// You can get your displayName by using the getUserInfo method;
const displayName = '22f5f84c-de9d-4ad6-97f2-201097b3b983';
const url =
    'https://connect.garmin.com/modern/proxy/wellness-service/wellness/dailyHeartRate/';
const dateString = '2020-03-24';
GCClient.get(url + displayName, { date: dateString });
```

and will net you the same result as using the provided way

```js
GCClient.getHeartRate();
```

Notice how the client will keep track of the url's, your user information as well as keeping the session alive.

## Limitations

Many responses from Garmin Connect are missing type definitions and defaults to `unknown`. Feel free to add types by opening a pull request.

For now, this library only supports the following:

-   Get user info
-   Get social user info
-   Get heart rate
-   Set body weight
-   Get list of workouts
-   Add new workouts
-   Add workouts to you calendar
-   Remove previously added workouts
-   Get list of activities
-   Get details about one specific activity
-   Get the step count
-   Get earned badges
-   Get available badges
-   Get details about one specific badge
