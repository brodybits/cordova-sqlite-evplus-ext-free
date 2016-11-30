# Cordova/PhoneGap sqlite storage - premium enterprise version with performance improvements for Android and FUTURE TBD other features

Native interface to sqlite in a Cordova/PhoneGap plugin for Android, iOS, macOS, and Windows 10 (UWP), with API similar to HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/).

This version uses a special native C library to provide significantly better performance as well as memory usage improvements for Android.

This version is available under GPL v3 (http://www.gnu.org/licenses/gpl.txt) or premium commercial license and includes components available under the MIT and Apache 2.0 licenses. Contact for commercial license: sales@litehelpers.net

**NOTICE:** Commercial licenses for [litehelpers / Cordova-sqlite-enterprise-free](https://github.com/litehelpers/Cordova-sqlite-enterprise-free) are **NOT** valid for this version. For an upgrade please contact <sales@litehelpers.net>.

TBD Circle CI (Android)/Travis CI (iOS)

## About this version

Premium enterprise version with performance improvements for Android and FUTURE TBD other features - common version branch with external sqlite3 dependencies installed by a before_plugin_install hook.

<!-- END About this version -->

## Services available

The primary author and maintainer [@brodybits (Christopher J. Brody aka Chris Brody)](https://github.com/brodybits) is available for part-time contract assignments. Services available for this project include:

- Support for Ionic and other Angular derivatives
- Single issue support
- Warranty and support retainers
- Priority fixes and enhancements
- Custom feature development

Other services available include:

- Front-end/back-end development
- Mentoring and training services

Fixed cost contract assignments are preferred.

For more information:
- <http://litehelpers.net/>
- <sales@litehelpers.net>

<!-- END Services available -->

## A quick tour

To open a database:

```Javascript
var db = null;

document.addEventListener('deviceready', function() {
  db = window.sqlitePlugin.openDatabase({name: 'demo.db', location: 'default'});
});
```

**IMPORTANT:** Like with the other Cordova plugins your application must wait for the `deviceready` event. This is especially tricky in Angular/ngCordova/Ionic controller/factory/service callbacks which may be triggered before the `deviceready` event is fired.

To populate a database using the standard transaction API:

```Javascript
  db.transaction(function(tx) {
    tx.executeSql('CREATE TABLE IF NOT EXISTS DemoTable (name, score)');
    tx.executeSql('INSERT INTO DemoTable VALUES (?,?)', ['Alice', 101]);
    tx.executeSql('INSERT INTO DemoTable VALUES (?,?)', ['Betty', 202]);
  }, function(error) {
    console.log('Transaction ERROR: ' + error.message);
  }, function() {
    console.log('Populated database OK');
  });
```

To check the data using the standard transaction API:

```Javascript
  db.transaction(function(tx) {
    tx.executeSql('SELECT count(*) AS mycount FROM DemoTable', [], function(tx, rs) {
      console.log('Record count (expected to be 2): ' + rs.rows.item(0).mycount);
    }, function(tx, error) {
      console.log('SELECT error: ' + error.message);
    });
  });
```

To populate a database using the SQL batch API:

```Javascript
  db.sqlBatch([
    'CREATE TABLE IF NOT EXISTS DemoTable (name, score)',
    [ 'INSERT INTO DemoTable VALUES (?,?)', ['Alice', 101] ],
    [ 'INSERT INTO DemoTable VALUES (?,?)', ['Betty', 202] ],
  ], function() {
    console.log('Populated database OK');
  }, function(error) {
    console.log('SQL batch ERROR: ' + error.message);
  });
```

To check the data using the single SQL statement API:

```Javascript
  db.executeSql('SELECT count(*) AS mycount FROM DemoTable', [], function(rs) {
    console.log('Record count (expected to be 2): ' + rs.rows.item(0).mycount);
  }, function(error) {
    console.log('SELECT SQL statement ERROR: ' + error.message);
  });
```

See the [Sample section](#sample) for a sample with a more detailed explanation.

<!-- END quick tour -->

## Status

- NOT supported by PhoneGap Developer App or PhoneGap Desktop App
- This version uses a `before_plugin_install` hook to install sqlite3 library dependencies from `cordova-sqlite-evcore-free-dependencies` via npm.
- A recent version of the Cordova CLI (such as `6.4.0`) is recommended. Possible issues with older Cordova versions include:
  - Cordova CLI older than `6.4.0` include an old version of cordova-ios that requires an extra `cordova prepare` step;
  - Cordova versions older than `6.0.0` are missing the `cordova-ios@4.0.0` security fixes.
- Use of other systems such as Cordova Plugman, PhoneGap CLI, PhoneGap Build, and Intel XDK is *not* supported since they do not honor the `before_plugin_install` hook. These are supported by: [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free)
- The iOS database location is now mandatory, as documented below.
- SQLite version `3.15.2` included with the following defines:
  - `SQLITE_TEMP_STORE=2`
  - `SQLITE_THREADSAFE=2`
  - `SQLITE_ENABLE_FTS3`
  - `SQLITE_ENABLE_FTS3_PARENTHESIS`
  - `SQLITE_ENABLE_FTS4`
  - `SQLITE_ENABLE_RTREE`
  - Not defined for Android: `SQLITE_ENABLE_COLUMN_METADATA`
  - iOS/macOS ONLY: `SQLITE_LOCKING_STYLE=1` `SQLITE_OMIT_BUILTIN_TEST` `SQLITE_OMIT_LOAD_EXTENSION`
  - Windows ONLY: `SQLITE_OS_WINRT`
  - **NOTE:** No default page/cache size is defined, newer default values are described at <http://sqlite.org/pgszchng2016.html>.
- This version supports the use of two (2) possible Android sqlite database implementations:
  - default: high-performance, lightweight [litehelpers / Android-sqlite-evcore-native-driver-free](https://github.com/litehelpers/Android-sqlite-evcore-native-driver-free) NDK library (C implementation)
  - optional: built-in Android database classes (usage described below)
- Windows 8.1 and Windows Phone 8.1 are no longer supported by this version. WP8, Windows 8.1, and Windows Phone 8.1 support is available in: [litehelpers / Cordova-sqlite-legacy-build-support](https://github.com/litehelpers/Cordova-sqlite-legacy-build-support)
- The following features are available in [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) (FUTURE TBD will be in Cordova-sqlite-evcore-extbuild-free along with the Android performance enhancements):
  - REGEXP support (Android/iOS)
  - Pre-populated database (Android/iOS/Windows)
- Windows version (using a customized version of the performant C++ [doo / SQLite3-WinRT](https://github.com/doo/SQLite3-WinRT) component) has the following known limitations:
  - Issue with UNICODE `\u0000` character (same as `\0`)
  - No background processing
  - INCORRECT error code (0) and INCONSISTENT error message (missing actual error info) in error callbacks ref: [litehelpers/Cordova-sqlite-storage#539](https://github.com/litehelpers/Cordova-sqlite-storage/issues/539)
  - Issue with emojis and other 4-octet UTF-8 characters (apparently not stored correctly) ref: [litehelpers/Cordova-sqlite-storage#564](https://github.com/litehelpers/Cordova-sqlite-storage/issues/564)
  - Not possible to read BLOB column values
  - Issue with `undefined` SQL parameter argument value described below
  - It is **not** possible to use this plugin with the default "Any CPU" target. A specific target CPU type **must** be specified when building an app with this plugin.
  - Windows version may crash in certain cases when invalid (bogus) function parameters are given, for example: `db.readTransaction('bogus');`
  - Hard coded dependency on platform toolset libraries included by Visual Studio 2015 ref: [litehelpers/Cordova-sqlite-storage#580](https://github.com/litehelpers/Cordova-sqlite-storage/issues/580)
- macOS version ("osx" platform) has not been tested in a release build and should be considered pre-alpha.
- FTS3, FTS4, and R-Tree support is tested working OK in this version (for all target platforms in this version branch Android/iOS/macOS/Windows)
- Android is supported back to SDK 10 (a.k.a. Gingerbread, Android 2.3.3); support for older versions is available upon request.
- iOS versions supported: 8.x/9.x/10.x (see [deviations section](#deviations) below for differences in case of WKWebView)
- Patches will NOT be accepted on this version due to possible licensing issues.

<!-- END Status -->

## Announcements

- [brodybits / sql-promise-helper](https://github.com/brodybits/sql-promise-helper) provides a Promise-based API wrapper.
- [nolanlawson / pouchdb-adapter-cordova-sqlite](https://github.com/nolanlawson/pouchdb-adapter-cordova-sqlite) supports this plugin along with other implementations such as [nolanlawson / sqlite-plugin-2](https://github.com/nolanlawson/sqlite-plugin-2) and [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql).
- macOS ("osx" platform) is now supported
- For the Android platform this version uses the high-performance and lightweight [litehelpers / Android-sqlite-evcore-native-driver-free](https://github.com/litehelpers/Android-sqlite-evcore-native-driver-free) C library for JSON and SQL statement handling and processes large batches in less than half the time compared to [litehelpers / Cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage), as measured by: [brodybits / Cordova-sql-test-app](https://github.com/brodybits/Cordova-sql-test-app)
- The [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) project is a CC0 (public domain) starting point to reproduce issues with this plugin and may be used as a quick way to start developing a new app.
- Published [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist) and [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls).
- Self-test functions to verify proper installation and operation of this plugin
- More explicit `openDatabase` and `deleteDatabase` `iosDatabaseLocation` option
- Added simple sql batch function
- [MetaMemoryT / websql-promise](https://github.com/MetaMemoryT/websql-promise) now provides a Promises-based interface to both Web SQL and this plugin
- iOS version is now fixed to override the correct pluginInitialize method and should work with recent versions of iOS

<!-- END Announcements -->

## Highlights

- Drop-in replacement for HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/): the only change should be to replace the static `window.openDatabase()` factory call with `window.sqlitePlugin.openDatabase()`, with parameters as documented below. Known deviations are documented in the [deviations section](#deviations) below.
- Failure-safe nested transactions with batch processing optimizations (according to HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/))
- API is designed to be as flexible as possible but does not allow the application to leave any transactions hanging open.
- As described in [this posting](http://brodyspark.blogspot.com/2012/12/cordovaphonegap-sqlite-plugins-offer.html):
  - Keeps sqlite database in a user data location that is known; can be reconfigured (iOS version); and may be synchronized to iCloud (iOS version).
  - No extra size limit. SQLite limits described at: <http://www.sqlite.org/limits.html>
- Also tested with multi-page applications
- This project is self-contained though with sqlite3 dependencies auto-fetched by npm. No dependencies on other plugins such as cordova-plugin-file
- Windows 8.1/Windows Phone 8.1/Windows 10 version uses a customized version of the performant C++ [doo / SQLite3-WinRT](https://github.com/doo/SQLite3-WinRT) component.
- [SQLCipher](https://www.zetetic.net/sqlcipher/) support for Android/iOS/Windows is available at: [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter)
- Intellectual property:
  - All source code is tracked to the original author in git
  - Major authors are tracked in AUTHORS.md
  - Licensing of each component is tracked in LICENSE.md
  - History of this project is also described in HISTORY.md

**TIP:** It is possible to migrate from Cordova to a pure native solution and continue using the data stored by this plugin.

<!-- END Highlights -->

## Getting started

### Recommended prerequisites

- Install a recent version of Cordova CLI, create a simple app with no plugins, and run it on the desired target platforms.
- Add a very simple plugin such as `cordova-plugin-dialogs` or an echo plugin and get it working. Ideally you should be able to handle a callback with some data coming from a prompt.

These prereqisites are very well documented in a number of excellent resources including:
- <http://cordova.apache.org/> (redirected from <http://cordova.io>)
- <http://www.tutorialspoint.com/cordova/>
- <https://ccoenraets.github.io/cordova-tutorial/>
- <https://www.toptal.com/mobile/developing-mobile-applications-with-apache-cordova>
- <http://www.tutorialspoint.com/cordova/index.htm>

More resources can be found by <https://www.google.com/search?q=cordova+tutorial>. There are even some tutorials available on YouTube as well.

In addition, this guide assumes a basic knowledge of some key JavaScript concepts such as variables, function calls, and callback functions.

**MAJOR TIPS:** As described in the [Installing](#installing) section:
- It is recommended to use the `--save` flag when installing plugins to track them in `config.xml`. If all plugins are tracked in `config.xml` then there is no need to commit the `plugins` subdirectory tree into the source repository.
- In general it is *not* recommended to commit the `platforms` subdirectory tree into the source repository.

**NOTICE:** This plugin is only supported with the Cordova CLI. This plugin is *not* supported with other Cordova/PhoneGap systems such as PhoneGap CLI, PhoneGap Build, Plugman, Intel XDK, Webstorm, etc.

### Windows platform notes

The Windows platform can present a number of challenges which increase when using this plugin. The following tips are recommended for getting started with Windows:

- First start to build and run an app on another platform such as Android or iOS with this plugin.
- Try working with a very simple app using simpler plugins such as cordova-plugin-dialogs and possibly cordova-plugin-file on the Windows platform.
- Read through the **Windows platform usage** of the [Installing](#installing) section.
- Then try adding this plugin to a very simple app such as [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) and running the Windows project in the Visual Studio GUI with a specific target CPU selected. **WARNING:** It is not possible to use this plugin with the "Any CPU" target.

### Quick installation

Use the following command to install this plugin from the Cordova CLI:

```shell
cordova plugin add https://github.com/litehelpers/Cordova-sqlite-evcore-common-free --save
```

Add any desired platform(s) if not already present, for example:

```shell
cordova platform add android
```

**OPTIONAL:** prepare before building (**MANDATORY** for cordova-ios older than 4.3.0 (Cordova CLI 6.4.0))

```shell
cordova prepare
```

or to prepare for a single platform, Android for example:

```shell
cordova prepare android
```

Please see the [Installing](#installing) section for more details.

**NOTE:** The new [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) project includes the echo test, self test, and string test described below along with some more sample functions.

<!-- END Quick installation -->

### Self test

Try the following programs to verify successful installation and operation:

**Echo test** - verify successful installation and build:

```js
document.addEventListener('deviceready', function() {
  window.sqlitePlugin.echoTest(function() {
    console.log('ECHO test OK');
  });
});
```

**Self test** - automatically verify basic database access operations including opening a database; basic CRUD operations (create data in a table, read the data from the table, update the data, and delete the data); close and delete the database:

```js
document.addEventListener('deviceready', function() {
  window.sqlitePlugin.selfTest(function() {
    console.log('SELF test OK');
  });
});
```

**NOTE:** It may be easier to use a JavaScript or native `alert` function call along with (or instead of) `console.log`  to verify that the installation passes both tests. Same for the SQL string test variations below. (Note that the Windows platform does not support the standard `alert` function, please use `cordova-plugin-dialogs` instead.)

### SQL string test

This test verifies that you can open a database, execute a basic SQL statement, and get the results (should be `TEST STRING`):

```js
document.addEventListener('deviceready', function() {
  var db = window.sqlitePlugin.openDatabase({name: 'test.db', location: 'default'});
  db.transaction(function(tr) {
    tr.executeSql("SELECT upper('Test string') AS upperString", [], function(tr, rs) {
      console.log('Got upperString result: ' + rs.rows.item(0).upperString);
    });
  });
});
```

Here is a variation that uses a SQL parameter instead of a string literal:

```js
document.addEventListener('deviceready', function() {
  var db = window.sqlitePlugin.openDatabase({name: 'test.db', location: 'default'});
  db.transaction(function(tr) {
    tr.executeSql('SELECT upper(?) AS upperString', ['Test string'], function(tr, rs) {
      console.log('Got upperString result: ' + rs.rows.item(0).upperString);
    });
  });
});
```

### Moving forward

It is recommended to read through the [usage](#usage) and [sample](#sample) sections before building more complex applications. In general it is recommended to start by doing things one step at a time, especially when an application does not work as expected.

The new [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) sample is intended to be a boilerplate to reproduce and demonstrate any issues you may have with this plugin. You may also use it as a starting point to build a new app.

In case you get stuck with something please read through the [support](#support) section and follow the instructions before raising an issue. Professional support is also available by contacting: <sales@litehelpers.net>

### Plugin examples and tutorials

- [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test)
- <http://stackoverflow.com/questions/26604952/a-simple-cordova-android-example-including-sqlite-read-write-and-search>


### SQLite resources

- <http://www.tutorialspoint.com/sqlite/index.htm> with a number of helpful articles

### Some other Cordova resources

- <http://www.tutorialspoint.com/cordova/cordova_file_system.htm>

<!-- END Getting started -->

## Some apps using this plugin

TBD *your app here*

<!-- END Some apps using this plugin -->

## Security

### Security of sensitive data

According to [Web SQL Database API 7.2 Sensitivity of data](https://www.w3.org/TR/webdatabase/#sensitivity-of-data):
>User agents should treat persistently stored data as potentially sensitive; it's quite possible for e-mails, calendar appointments, health records, or other confidential documents to be stored in this mechanism.
>
>To this end, user agents should ensure that when deleting data, it is promptly deleted from the underlying storage.

Unfortunately this plugin will not actually overwrite the deleted content unless the [secure_delete PRAGMA](https://www.sqlite.org/pragma.html#pragma_secure_delete) is used.

### SQL injection

As "strongly recommended" by [Web SQL Database API 8.5 SQL injection](https://www.w3.org/TR/webdatabase/#sql-injection):
>Authors are strongly recommended to make use of the `?` placeholder feature of the `executeSql()` method, and to never construct SQL statements on the fly.

<!-- END Security -->

# Avoiding data loss

- Double-check that the application code follows the documented API for SQL statements, parameter values, success callbacks, and error callbacks.
- For standard Web SQL transactions include a transaction error callback with the proper logic that indicates to the user if data cannot be stored for any reason. In case of individual SQL error handlers be sure to indicate to the user if there is any issue with storing data.
- For single statement and batch transactions include an error callback with logic that indicates to the user if data cannot be stored for any reason.

<!-- Avoiding data loss -->

## Deviations

### Some known deviations from the Web SQL database standard

- The `window.sqlitePlugin.openDatabase` static factory call takes a different set of parameters than the standard Web SQL `window.openDatabase` static factory call. In case you have to use existing Web SQL code with no modifications please see the **Web SQL replacement tip** below.
- This plugin does *not* support the database creation callback or standard database versions. Please read the **Database schema versions** section below for tips on how to support database schema versioning.
- This plugin does *not* support the synchronous Web SQL interfaces.
- Known issues with handling of certain ASCII/UNICODE characters as described below.
- It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others ref: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)
- It is possible to insert multiple rows like: `transaction.executeSql('INSERT INTO MyTable VALUES (?,?),(?,?)', ['Alice', 101, 'Betty', 102]);` which was not supported by SQLite 3.6.19 as referenced by [Web SQL API section 5](https://www.w3.org/TR/webdatabase/#web-sql). The iOS WebKit Web SQL implementation seems to support this as well.
- Unlike the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) this plugin handles executeSql calls with too few parameters without error reporting. In case of too many parameters this plugin reports error code 0 (SQLError.UNKNOWN_ERR) while Android/iOS (WebKit) Web SQL correctly reports error code 5 (SQLError.SYNTAX_ERR) ref: https://www.w3.org/TR/webdatabase/#dom-sqlexception-code-syntax
- Issues with positive and negative `Infinity` SQL parameter argument values are treated like `null` by this plugin on Android and iOS ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)
- Positive and negative `Infinity` result values cause a crash on iOS/macOS (ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)) or a hanging transaction queue on Android
- Windows version fails with an error in case an `undefined` SQL parameter argument value is used.
- Known issue(s) with of certain ASCII/UNICODE characters as described below.
- Boolean `true` and `false` values are handled by converting them to the "true" and "false" TEXT string values, same as WebKit Web SQL on Android and iOS. This does not seem to be 100% correct as discussed in: [litehelpers/Cordova-sqlite-storage#545](https://github.com/litehelpers/Cordova-sqlite-storage/issues/545)
- A number of uncategorized errors such as CREATE VIRTUAL TABLE USING bogus module are reported with error code 5 (SQLError.SYNTAX_ERR) on Android/iOS/macOS by both (WebKit) Web SQL and this plugin.
- Issues with error code on Windows as well as Android with the `androidDatabaseImplementation: 2` setting described below.
- In case of an issue that causes an API function to throw an exception (Android/iOS WebKit) Web SQL includes includes a code member with value of 0 (SQLError.UNKNOWN_ERR) in the exception while the plugin includes no such code member.
- This plugin supports some non-standard features as documented below.
- Results of SELECT with BLOB data such as `SELECT LOWER(X'40414243') AS myresult`, `SELECT X'40414243' AS myresult`, or reading data stored by `INSERT INTO MyTable VALUES (X'40414243')` are not consistent on Android in case the built-in Android database is used (using the `androidDatabaseImplementation: 2` setting in `window.sqlitePlugin.openDatabase`) or Windows. (These work with Android/iOS WebKit Web SQL and have been supported by SQLite for a number of years.)
- Whole number parameter argument values such as `42`, `-101`, or `1234567890123` are handled as INTEGER values by this plugin on Android, iOS (default UIWebView), and Windows while they are handled as REAL values by (WebKit) Web SQL and this plugin on macOS or iOS with WKWebView. This is evident in certain test operations such as `SELECT ? as myresult` or `SELECT TYPEOF(?) as myresult` and storage in a field with TEXT affinity.
- INTEGER, REAL, +/- `Infinity`, `NaN`, `null`, `undefined` parameter argument values are handled as TEXT string values on Android in case the built-in Android database (`androidDatabaseImplementation: 2` setting) is used. (This is evident in certain test operations such as `SELECT ? as myresult` or `SELECT TYPEOF(?) as myresult` and storage in a field with TEXT affinity.)
- In case of invalid transaction callback arguments such as string values the plugin attempts to execute the transaction while (WebKit) Web SQL would throw an exception. (This may cause the plugin to crash on Windows in case of a read-only transaction.)
- The plugin handles invalid SQL arguments array values such as `false`, `true`, or a string as if there were no arguments while (WebKit) Web SQL would throw an exception. NOTE: In case of a function in place of the SQL arguments array WebKit Web SQL would report a transaction error while the plugin would simply ignore the function.
- In case of invalid SQL callback arguments such as string values the plugin may execute the SQL and signal transaction success or failure while (WebKit) Web SQL would throw an exception.
- In certain cases such as `transaction.executeSql(null)` or `transaction.executeSql(undefined)` the plugin throws an exception while (WebKit) Web SQL indicates a transaction failure.
- In certain cases such as `transaction.executeSql()` with no arguments (Android/iOS WebKit) Web SQL includes includes a code member with value of 0 (SQLError.UNKNOWN_ERR) in the exception while the plugin includes no such code member.
- If the SQL arguments are passed in an `Array` subclass object where the `constructor` does not point to `Array` then the SQL arguments are ignored by the plugin.
- The results data objects are not immutable as specified/implied by [Web SQL API section 4.5](https://www.w3.org/TR/webdatabase/#database-query-results).

### Security of deleted data

See **Security of sensitive data** in the [Security](#security) section above.

### Other differences with WebKit Web SQL implementations

- In case of a SQL error handler that does not recover the transaction, WebKit Web SQL (Android/iOS) would incorrectly report error code 0 while the plugin would report the same error code as in the SQL error handler. (In case of an error with no SQL error handler then Android/iOS WebKit Web SQL would report the same error code that would have been reported in the SQL error hander.)
- In case a transaction function throws an exception, the message and code if present are reported by the plugin but *not* by (WebKit) Web SQL.
- SQL error messages are inconsistent on Windows and less descriptive on Android in case of the default Android-sqlite-evcore-native-driver-free implementation.
- There are some other differences in the SQL error messages reported by WebKit Web SQL and this plugin.

<!-- END Deviations -->

## Known issues

- iOS/macOS version does not support certain rapidly repeated open-and-close or open-and-delete test scenarios due to how the implementation handles background processing
- As described below, auto-vacuum is NOT enabled by default.
- The Android and Windows versions do not always handle four-byte UTF-8 characters emoji characters such as `\u1F603` (SMILING FACE, MOUTH OPEN) correctly ref: [litehelpers/Cordova-sqlite-storage#564](https://github.com/litehelpers/Cordova-sqlite-storage/issues/564). It is sometimes possible to store and retrieve such characters but certain operations hex conversions do not work properly with the default Android-sqlite-evcore-native-driver-free implementation. It is suspected that such characters would be stored incorrectly in the Android and Windows versions. This is not an issue in case the built-in Android database is used (using the `androidDatabaseImplementation: 2` setting in `window.sqlitePlugin.openDatabase`).
- It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others ref: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)
- INSERT statement that affects multiple rows (due to SELECT cause or using TRIGGER(s), for example) reports incorrect rowsAffected on Android in case the built-in Android database used (using the `androidDatabaseImplementation` option in `window.sqlitePlugin.openDatabase`)
- Memory issue observed when adding a large number of records due to the JSON implementation which is improved in [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (available with GPL or commercial license options)
- Infinity (positive or negative) values are not supported on Android/iOS/macOS due to issues described above including a possible crash on iOS/macOS (ref: [litehelpers/Cordova-sqlite-storage#405](https://github.com/litehelpers/Cordova-sqlite-storage/issues/405)) or a hanging transaction queue on Android.
- A stability issue was reported on the iOS version when in use together with [SockJS](http://sockjs.org/) client such as [pusher-js](https://github.com/pusher/pusher-js) at the same time (see [litehelpers/Cordova-sqlite-storage#196](https://github.com/litehelpers/Cordova-sqlite-storage/issues/196)). The workaround is to call sqlite functions and [SockJS](http://sockjs.org/) client functions in separate ticks (using setTimeout with 0 timeout).
- SQL errors are reported with an INCORRECT error code (0) on Windows ref: [litehelpers/Cordova-sqlite-storage#539](https://github.com/litehelpers/Cordova-sqlite-storage/issues/539). In certain cases SQL errors are also reported with error code 0 on Android in case the built-in Android database is used (using the `androidDatabaseImplementation: 2` setting in `window.sqlitePlugin.openDatabase`).
- Possible crash on Android when using Unicode emoji and other 4-octet UTF-8 characters due to [Android bug 81341](https://code.google.com/p/android/issues/detail?id=81341), which *should* be fixed in Android 6.
- Close/delete database bugs described below.
- The default Android implementation does not currently handle vertical tab, form feed, or backspace characters properly.
- When a database is opened and deleted without closing, the iOS/macOS version is known to leak resources.
- It is NOT possible to open multiple databases with the same name but in different locations (iOS/macOS).
- Incorrect or missing rowsAffected in results for INSERT/UPDATE/DELETE SQL statements with extra semicolon(s) in the beginning for Android in case the `androidDatabaseImplementation: 2` (built-in android.database implementation) option is used.

Some more known issues are tracked in the [open Cordova-sqlite-storage bugs](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Abug).

<!-- END Known issues -->

## Other limitations

- ~~The db version, display name, and size parameter values are not supported and will be ignored.~~ (No longer supported by the API)
- Absolute and relative subdirectory path(s) are not tested or supported.
- This plugin will not work before the callback for the 'deviceready' event has been fired, as described in **Usage**. (This is consistent with the other Cordova plugins.)
- This plugin cannot support large records. TBD: specify maximum record; FUTURE TBD: to be fixed in [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) (available with GPL or commercial license options)
- This version will not work within a web worker (not properly supported by the Cordova framework). Use within a web worker is supported for Android and iOS in: [litehelpers / Cordova-sqlite-evplus-legacy-workers-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-workers-free) (available with GPL or premium commercial license options)
- In-memory database `db=window.sqlitePlugin.openDatabase({name: ':memory:', ...})` is currently not supported.
- The Android version cannot work with more than 100 open db files (due to the threading model used).
- SQL error messages on Windows version are not consistent with Android/iOS/macOS versions.
- UNICODE `\u2028` (line separator) and `\u2029` (paragraph separator) characters are currently not supported and known to be broken in iOS, macOS, and Android version due to JSON issues reported in [Cordova bug CB-9435](https://issues.apache.org/jira/browse/CB-9435) and [cordova/cordova-discuss#57](https://github.com/cordova/cordova-discuss/issues/57). There *may* be a similar issue with certain other UNICODE characters in the iOS/macOS version (needs further investigation). This is fixed for iOS in: [litehelpers / Cordova-sqlite-evplus-legacy-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-free) and [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) (available with GPL or special commercial license options) as well as [litehelpers / Cordova-sqlite-evplus-legacy-workers-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-workers-free) (available with GPL or premium commercial license options)
- BLOB type is not supported in this version branch (*reading* of BLOBs is supported by [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) for Android/iOS)
- UNICODE `\u0000` (same as `\0`) character not working on Android (default Android-sqlite-evcore-native-driver-free implementation) or Windows
- Case-insensitive matching and other string manipulations on Unicode characters, which is provided by optional ICU integration in the sqlite source and working with recent versions of Android, is not supported for any target platforms.
- iOS/macOS version uses a thread pool but with only one thread working at a time due to "synchronized" database access
- Extremely large query result may be slow due to the JSON implementation.
- ATTACH to another database file is not supported by this version. Attach/detach is supported (along with the memory and iOS UNICODE `\u2028` line separator / `\u2029` paragraph separator fixes) in: [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) (available with GPL or special commercial license options)
- UPDATE/DELETE with LIMIT or ORDER BY is not supported.
- WITH clause is not supported by older Android versions in case the `androidDatabaseImplementation: 2` (built-in android.database implementation) option is used.
- User-defined savepoints are not supported and not expected to be compatible with the transaction locking mechanism used by this plugin. In addition, the use of BEGIN/COMMIT/ROLLBACK statements is not supported.
- Problems have been reported when using this plugin with Crosswalk (for Android). It may help to install Crosswalk as a plugin instead of using Crosswalk to create the project.
- Does not work with [axemclion / react-native-cordova-plugin](https://github.com/axemclion/react-native-cordova-plugin) since the `window.sqlitePlugin` object is *not* properly exported (ES5 feature). It is recommended to use [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) for SQLite database access with React Native Android/iOS instead.

Some more limitations are tracked in the [open Cordova-sqlite-storage documentation issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Adocumentation).

<!-- END Other limitations -->

## Further testing needed

- Integration with PhoneGap developer app
- Use within [InAppBrowser](http://docs.phonegap.com/en/edge/cordova_inappbrowser_inappbrowser.md.html)
- Use within an iframe (see [litehelpers/Cordova-sqlite-storage#368 (comment)](https://github.com/litehelpers/Cordova-sqlite-storage/issues/368#issuecomment-154046367))
- Date/time handling
- Maximum record size supported
- Actual behavior when using SAVEPOINT(s)
- R-Tree is not fully tested with Android
- UNICODE characters not fully tested
- ORDER BY random() (ref: [litehelpers/Cordova-sqlite-storage#334](https://github.com/litehelpers/Cordova-sqlite-storage/issues/334))
- UPDATE/DELETE with LIMIT or ORDER BY (newer Android/iOS versions)
- Integration with JXCore for Cordova (must be built without sqlite(3) built-in)
- Delete an open database inside a statement or transaction callback.
- WITH clause (not supported by some older sqlite3 versions)
- Handling of invalid transaction and transaction.executeSql arguments
- Use of database locations on macOS
- Use of very large REAL numbers such as 1234567890123.4
- More emojis and other 4-octet UTF-8 characters
- `?NNN`/`:AAA`/`@AAAA`/`$AAAA` parameter placeholders ref: <https://www.sqlite.org/lang_expr.html#varparam>, <https://www.sqlite.org/c3ref/bind_blob.html>)
- Single-statement and SQL batch transaction calls with invalid arguments (TBD behavior subject to change)

<!-- END Further testing needed -->

## Some tips and tricks

- If you run into problems and your code follows the asynchronous HTML5/[Web SQL](http://www.w3.org/TR/webdatabase/) transaction API, you can try opening a test database using `window.openDatabase` and see if you get the same problems.
- In case your database schema may change, it is recommended to keep a table with one row and one column to keep track of your own schema version number. It is possible to add it later. The recommended schema update procedure is described below.

<!-- END Some tips and tricks -->

## Pitfalls

### Some common pitfall(s)

- If a database is opened using the standard `window.openDatabase` call it will not have any of the benefits of this plugin and features such as the `sqlBatch` call would not be available.
- It is NOT allowed to execute sql statements on a transaction that has already finished, as described below. This is consistent with the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/).
- The plugin class name starts with "SQL" in capital letters, but in Javascript the `sqlitePlugin` object name starts with "sql" in small letters.
- Attempting to open a database before receiving the 'deviceready' event callback.
- Inserting STRING into ID field
- Auto-vacuum is NOT enabled by default. It is recommended to periodically VACUUM the database.
- Transactions on a database are run sequentially. A large transaction could block smaller transactions requested afterwards.

### Some weird pitfall(s)

- intent whitelist: blocked intent such as external URL intent *may* cause this and perhaps certain Cordova plugin(s) to misbehave (see [litehelpers/Cordova-sqlite-storage#396](https://github.com/litehelpers/Cordova-sqlite-storage/issues/396))

### Angular/ngCordova/Ionic-related pitfalls

- Angular/ngCordova/Ionic controller/factory/service callbacks may be triggered before the 'deviceready' event is fired
- As discussed in [litehelpers/Cordova-sqlite-storage#355](https://github.com/litehelpers/Cordova-sqlite-storage/issues/355), it may be necessary to install ionic-plugin-keyboard

### Windows platform pitfalls

- This plugin does **not** work with the default "Any CPU" target. A specific, valid CPU target platform **must** be specified.
- It is **not** allowed to change the app ID in the Windows platform project. As described in the **Windows platform usage** of the [Installing](#installing) section a Windows-specific app ID may be declared using the `windows-identity-name` attribute or "WindowsStoreIdentityName" setting.
- A problem locating `SQLite3.md` generally means that there was a problem building the C++ library.

### General Cordova pitfalls

Documented in: [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls)

### General SQLite pitfalls

From <https://www.sqlite.org/datatype3.html#section_1>:
> SQLite uses a more general dynamic type system.

This is generally nice to have, especially in conjunction with a dynamically typed language such as JavaScript. Here are some major SQLite data typing principles:
- From <https://www.sqlite.org/datatype3.html#section_3>: the CREATE TABLE SQL statement declares each column with one of the following type affinities: TEXT, NUMERIC, INTEGER, REAL, or BLOB.
- From <https://www.sqlite.org/datatype3.html#section_3_1> with column type affinity determination rules: it should be possible to do CREATE TABLE with columns of almost any type name (for example: `CREATE TABLE MyTable (data ABC);`) and each column type affinity is determined according to pattern matching. If a declared column type name does not match any of the patterns the column has NUMERIC affinity.
- From <https://www.sqlite.org/datatype3.html#section_3_2>: a column with no data type name specified actually gets the BLOB affinity.

However there are some possible gotchas:

1. From <https://www.sqlite.org/datatype3.html#section_3_2>:
> Note that a declared type of "FLOATING POINT" would give INTEGER affinity, not REAL affinity, due to the "INT" at the end of "POINT". And the declared type of "STRING" has an affinity of NUMERIC, not TEXT.

2. From ibid: a column declared as "DATETIME" has NUMERIC affinity, which gives no hint whether an INTEGER Unix time value, a REAL Julian time value, or possibly even a TEXT ISO8601 date/time string may be stored (further refs:  <https://www.sqlite.org/datatype3.html#section_2_2>, <https://www.sqlite.org/datatype3.html#section_3>)

From <https://groups.google.com/forum/#!topic/phonegap/za7z51_fKRw>, as discussed in [litehelpers/Cordova-sqlite-storage#546](https://github.com/litehelpers/Cordova-sqlite-storage/issues/546): it was discovered that are some more points of possible confusion with date/time. For example, there is also a `datetime` function that returns date/time in TEXT string format. This should be considered a case of "DATETIME" overloading since SQLite is *not* case sensitive. This could really become confusing if different programmers or functions consider date/time to be stored in different ways.

FUTURE TBD: Proper date/time handling will be further tested and documented at some point.

<!-- END pitfalls -->

## Major TODOs

- More formal documentation of API, especially for non-standard functions
- IndexedDBShim adapter
- Version with proper BLOB support
- Further cleanup of [support](#support) section
- Improve [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) to be a better app template
- Add starter project with pre-populated database
- Resolve or document remaining [open Cordova-sqlite-storage bugs](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Abug)
- Resolve [open Cordova-sqlite-storage documentation issues](https://github.com/litehelpers/Cordova-sqlite-storage/issues?q=is%3Aissue+is%3Aopen+label%3Adocumentation)

## For future considertion

- Auto-vacuum option
- Browser platform
- Integrate with some other libraries such as Sequelize, Squel.js, WebSqlSync, Persistence.js, Knex, etc.

## Alternatives

### Other versions

- [litehelpers / Cordova-sqlite-evcore-extbuild-free](https://github.com/litehelpers/Cordova-sqlite-evcore-extbuild-free) - Android performance enhancements and support for arbitrary Android database location with all dependencies included, support for PhoneGap CLI/PhoneGap Build/plugman/Intel XDK. (GPL or commercial license options)
- [litehelpers / Cordova-sqlite-evplus-legacy-workers-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-workers-free) - version with support for web workers (Android/iOS), includes internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (iOS) (GPL or premium commercial license options)
- [litehelpers / Cordova-sqlite-evplus-legacy-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-free) - internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (iOS) (GPL or special commercial license options)
- [litehelpers / Cordova-sqlite-evplus-legacy-attach-detach-free](https://github.com/litehelpers/Cordova-sqlite-evplus-legacy-attach-detach-free) - version with support for ATTACH, includes internal memory improvements to support larger transactions (Android/iOS) and fix to support all Unicode characters (GPL or special commercial license options)
- [litehelpers / Cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage) - Cordova sqlite storage plugin _for Android/iOS/macOS/Windows 10_ with permissive licensing terms
- [litehelpers / cordova-sqlite-ext](https://github.com/litehelpers/cordova-sqlite-ext) - version with REGEXP support for Android/macOS/iOS and pre-populated database support for Android/iOS/macOS/Windows (permissive licensing terms)
- [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter) - supports [SQLCipher](https://www.zetetic.net/sqlcipher/) for Android/iOS/macOS/Windows (permissive licensing terms)
- [litehelpers / Cordova-sqlite-legacy-build-support](https://github.com/litehelpers/Cordova-sqlite-legacy-build-support) - maintenance of WP8 version along with the other supported platforms Android/iOS/_macOS/Windows 8.1/Windows Phone 8.1/Windows 10_; limited support for PhoneGap CLI/PhoneGap Build/plugman/Intel XDK; limited testing; limited updates (permissive licensing terms)
- Adaptation for React Native Android and iOS: [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) (permissive licensing terms)
- Original version for iOS (with a slightly different transaction API): [davibe / Phonegap-SQLitePlugin](https://github.com/davibe/Phonegap-SQLitePlugin) (permissive licensing terms)

<!-- END Other versions -->

### Other SQLite adapter projects

- [object-layer / AnySQL](https://github.com/object-layer/anysql) - Unified SQL API over multiple database engines
- [samikrc / CordovaSQLite](https://github.com/samikrc/CordovaSQLite) - Simpler sqlite plugin with a simpler API and browser platform
- [nolanlawson / sqlite-plugin-2](https://github.com/nolanlawson/sqlite-plugin-2) - Simpler fork/rewrite
- [nolanlawson / node-websql](https://github.com/nolanlawson/node-websql) - Web SQL API implementation for Node.js
- [an-rahulpandey / cordova-plugin-dbcopy](https://github.com/an-rahulpandey/cordova-plugin-dbcopy) - Alternative way to copy pre-populated database
- [EionRobb / phonegap-win8-sqlite](https://github.com/EionRobb/phonegap-win8-sqlite) - WebSQL add-on for Win8/Metro apps (perhaps with a different API), using an old version of the C++ library from [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) (as referenced by [01org / cordova-win8](https://github.com/01org/cordova-win8))
- [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) - C++ component that provides a nice SQLite API with promises for WinJS
- [01org / cordova-win8](https://github.com/01org/cordova-win8) - old, unofficial version of Cordova API support for Windows 8 Metro that includes an old version of the C++ [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT)
- [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql) - Windows 8(+) and Windows Phone 8(+) WebSQL plugin versions in C#
- [Thinkwise / cordova-plugin-websql](https://github.com/Thinkwise/cordova-plugin-websql) - fork of [Microsoft / cordova-plugin-websql](https://github.com/Microsoft/cordova-plugin-websql) that supports asynchronous execution
- [MetaMemoryT / websql-client](https://github.com/MetaMemoryT/websql-client) - provides the same API and connects to [websql-server](https://github.com/MetaMemoryT/websql-server) through WebSockets.

<!-- END Other SQLite adapter projects -->

### Alternative solutions

- Use [phearme / cordova-ContentProviderPlugin](https://github.com/phearme/cordova-ContentProviderPlugin) to query content providers on Android devices
- [ABB-Austin / cordova-plugin-indexeddb-async](https://github.com/ABB-Austin/cordova-plugin-indexeddb-async) - Asynchronous IndexedDB plugin for Cordova that uses [axemclion / IndexedDBShim](https://github.com/axemclion/IndexedDBShim) (Browser/iOS/Android/Windows) and [Thinkwise / cordova-plugin-websql](https://github.com/Thinkwise/cordova-plugin-websql) - (Windows)
- Another sqlite binding for React-Native (iOS version): [almost/react-native-sqlite](https://github.com/almost/react-native-sqlite)
- Use [NativeScript](https://www.nativescript.org) with its web view and [NathanaelA / nativescript-sqlite](https://github.com/Natha
naelA/nativescript-sqlite) (Android and/or iOS)
- Standard HTML5 [local storage](https://en.wikipedia.org/wiki/Web_storage#localStorage)
- [Realm.io](https://realm.io/)

<!-- END Alternative solutions -->

# Usage

## Self-test functions

To verify that both the Javascript and native part of this plugin are installed in your application:

```js
window.sqlitePlugin.echoTest(successCallback, errorCallback);
```

To verify that this plugin is able to open a database (named `___$$$___litehelpers___$$$___test___$$$___.db`), execute the CRUD (create, read, update, and delete) operations, and clean it up properly:

```js
window.sqlitePlugin.selfTest(successCallback, errorCallback);
```

**IMPORTANT:** Please wait for the 'deviceready' event (see below for an example).

## General

- Drop-in replacement for HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/): the only change should be to replace the static `window.openDatabase()` factory call with `window.sqlitePlugin.openDatabase()`, with parameters as documented below. Some other known deviations are documented below. Please report if you find any other possible deviations.
- Single-page application design is recommended.
- In case of a multi-page application the JavaScript used by each page must use `sqlitePlugin.openDatabase` to open the database access handle object before it can access the data.

**NOTE:** If a sqlite statement in a transaction fails with an error, the error handler *must* return `false` in order to recover the transaction. This is correct according to the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) standard. This is different from the WebKit implementation of Web SQL in Android and iOS which recovers the transaction if a sql error hander returns a non-`true` value.

See the [Sample section](#sample) for a sample with detailed explanations.

## Opening a database

To open a database access handle object (in the **new** default location):

```js
var db = window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default'}, successcb, errorcb);
```

**WARNING:** The new "default" location value is *NOT* the same as the old default location and would break an upgrade for an app that was using the old default value (0) on iOS.

To specify a different location (affects iOS/macOS *only*):

```js
var db = window.sqlitePlugin.openDatabase({name: 'my.db', iosDatabaseLocation: 'Library'}, successcb, errorcb);
```

where the `iosDatabaseLocation` option may be set to one of the following choices:
- `default`: `Library/LocalDatabase` subdirectory - *NOT* visible to iTunes and *NOT* backed up by iCloud
- `Library`: `Library` subdirectory - backed up by iCloud, *NOT* visible to iTunes
- `Documents`: `Documents` subdirectory - visible to iTunes and backed up by iCloud

**WARNING:** Again, the new "default" iosDatabaseLocation value is *NOT* the same as the old default location and would break an upgrade for an app using the old default value (0) on iOS.

*ALTERNATIVE (deprecated):*
- `var db = window.sqlitePlugin.openDatabase({name: "my.db", location: 1}, successcb, errorcb);`

with the `location` option set to one the following choices (affects iOS *only*):
- `0` ~~(default)~~: `Documents` - visible to iTunes and backed up by iCloud
- `1`: `Library` - backed up by iCloud, *NOT* visible to iTunes
- `2`: `Library/LocalDatabase` - *NOT* visible to iTunes and *NOT* backed up by iCloud (same as using "default")

No longer supported (see tip below to overwrite `window.openDatabase`): ~~`var db = window.sqlitePlugin.openDatabase("myDatabase.db", "1.0", "Demo", -1);`~~

**IMPORTANT:** Please wait for the 'deviceready' event, as in the following example:

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default'});
  // ...
}
```

The successcb and errorcb callback parameters are optional but can be extremely helpful in case anything goes wrong. For example:

```js
window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default'}, function(db) {
  db.transaction(function(tx) {
    // ...
  }, function(err) {
    console.log('Open database ERROR: ' + JSON.stringify(err));
  });
});
```

If any sql statements or transactions are attempted on a database object before the openDatabase result is known, they will be queued and will be aborted in case the database cannot be opened.

**OTHER NOTES:**
- The database file name should include the extension, if desired.
- It is possible to open multiple database access handle objects for the same database.
- The database handle access object can be closed as described below.

**Web SQL replacement tip:**

To overwrite `window.openDatabase`:

```Javascript
window.openDatabase = function(dbname, ignored1, ignored2, ignored3) {
  return window.sqlitePlugin.openDatabase({name: dbname, location: 'default'});
};
```

### iCloud backup notes

As documented in the "**A User’s iCloud Storage Is Limited**" section of [iCloudFundamentals in Mac Developer Library iCloud Design Guide](https://developer.apple.com/library/mac/documentation/General/Conceptual/iCloudDesignGuide/Chapters/iCloudFundametals.html) (near the beginning):

<blockquote>
<ul>
<li><b>DO</b> store the following in iCloud:
  <ul>
   <li>[<i>other items omitted</i>]</li>
   <li>Change log files for a SQLite database (a SQLite database’s store file must never be stored in iCloud)</li>
  </ul>
</li>
<li><b>DO NOT</b> store the following in iCloud:
  <ul>
   <li>[<i>items omitted</i>]</li>
  </ul>
</li>
</ul>
- <cite><a href="https://developer.apple.com/library/mac/documentation/General/Conceptual/iCloudDesignGuide/Chapters/iCloudFundametals.html">iCloudFundamentals in Mac Developer Library iCloud Design Guide</a>
</blockquote>

#### How to disable iCloud backup

Use the `location` or `iosDatabaseLocation` option in `sqlitePlugin.openDatabase()` to store the database in a subdirectory that is *NOT* backed up to iCloud, as described in the section below.

**NOTE:** Changing `BackupWebStorage` in `config.xml` has no effect on a database created by this plugin. `BackupWebStorage` applies only to local storage and/or Web SQL storage created in the WebView (*not* using this plugin). For reference: [phonegap/build#338 (comment)](https://github.com/phonegap/build/issues/338#issuecomment-113328140)

<!-- END iCloud backup notes -->

### Android sqlite implementation

By default, this plugin uses [litehelpers / Android-sqlite-evcore-native-driver-free](https://github.com/litehelpers/Android-sqlite-evcore-native-driver-free) with the performance and memory usage enhancements. To use the built-in Android database classes instead:

```js
var db = window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default', androidDatabaseImplementation: 2});
```

<!-- END Android sqlite implementation -->

### Workaround for Android db locking issue

[litehelpers/Cordova-sqlite-storage#193](https://github.com/litehelpers/Cordova-sqlite-storage/issues/193) was reported (as observed by a number of app developers) that when using the `androidDatabaseImplementation: 2` setting on certain Android versions and if the app is stopped or aborted without closing the database then:
- (sometimes) there is an unexpected database lock
- the data that was inserted is lost.

The cause of this issue remains unknown. Of interest: [android / platform_external_sqlite commit d4f30d0d15](https://github.com/android/platform_external_sqlite/commit/d4f30d0d1544f8967ee5763c4a1680cb0553039f) which references and includes the sqlite commit at: http://www.sqlite.org/src/info/6c4c2b7dba

This is *not* an issue when the default [litehelpers / Android-sqlite-evcore-native-driver-free](https://github.com/litehelpers/Android-sqlite-evcore-native-driver-free) database implementation is used, which is the case when no `androidDatabaseImplementation` setting is used.

There is an optional workaround that simply closes and reopens the database file at the end of every transaction that is committed. The workaround is enabled by opening the database with options as follows:

```js
var db = window.sqlitePlugin.openDatabase({
  name: 'my.db',
  location: 'default',
  androidDatabaseImplementation: 2,
  androidLockWorkaround: 1
});
```

**IMPORTANT NOTE:** This workaround is *only* applied when using `db.sqlBatch` or `db.transaction()`, *not* applied when running `executeSql()` on the database object.

<!-- Workaround for Android db locking issue -->

## SQL transactions

The following types of SQL transactions are supported by this version:
- Single-statement transactions
- SQL batch transactions
- Standard asynchronous transactions

**NOTE:** Transaction requests are kept in one queue per database and executed in sequential order, according to the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/).

**WARNING:** - It is possible to request a SQL statement list such as "SELECT 1; SELECT 2" within a single SQL statement string, however the plugin will only execute the first statement and silently ignore the others. This could result in data loss if such a SQL statement list with any INSERT or UPDATE statement(s) are included. For reference: [litehelpers/Cordova-sqlite-storage#551](https://github.com/litehelpers/Cordova-sqlite-storage/issues/551)

### Single-statement transactions

Sample with INSERT:

```Javascript
db.executeSql('INSERT INTO MyTable VALUES (?)', ['test-value'], function (resultSet) {
  console.log('resultSet.insertId: ' + resultSet.insertId);
  console.log('resultSet.rowsAffected: ' + resultSet.rowsAffected);
}, function(error) {
  console.log('SELECT error: ' + error.message);
});
```

Sample with SELECT:

```Javascript
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (resultSet) {
  console.log('got stringlength: ' + resultSet.rows.item(0).stringlength);
}, function(error) {
  console.log('SELECT error: ' + error.message);
});
```

**NOTE/minor bug:** The object returned by `resultSet.rows.item(rowNumber)` is **not** immutable. In addition, multiple calls to `resultSet.rows.item(rowNumber)` with the same `rowNumber` on the same `resultSet` object return the same object. For example, the following code will show `Second uppertext result: ANOTHER`:

```Javascript
db.executeSql("SELECT UPPER('First') AS uppertext", [], function (resultSet) {
  var obj1 = resultSet.rows.item(0);
  obj1.uppertext = 'ANOTHER';
  console.log('Second uppertext result: ' + resultSet.rows.item(0).uppertext);
  console.log('SELECT error: ' + error.message);
});
```

<!-- END Single-statement transactions -->

### SQL batch transactions

Sample:

```Javascript
db.sqlBatch([
  'DROP TABLE IF EXISTS MyTable',
  'CREATE TABLE MyTable (SampleColumn)',
  [ 'INSERT INTO MyTable VALUES (?)', ['test-value'] ],
], function() {
  db.executeSql('SELECT * FROM MyTable', [], function (resultSet) {
    console.log('Sample column value: ' + resultSet.rows.item(0).SampleColumn);
  });
}, function(error) {
  console.log('Populate table error: ' + error.message);
});
```

In case of an error, all changes in a sql batch are automatically discarded using ROLLBACK.

<!-- END SQL batch transactions -->

### Standard asynchronous transactions

Standard asynchronous transactions follow the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) which is very well documented and uses BEGIN and COMMIT or ROLLBACK to keep the transactions failure-safe. Here is a simple example:

```Javascript
db.transaction(function(tx) {
  tx.executeSql('DROP TABLE IF EXISTS MyTable');
  tx.executeSql('CREATE TABLE MyTable (SampleColumn)');
  tx.executeSql('INSERT INTO MyTable VALUES (?)', ['test-value'], function(tx, resultSet) {
    console.log('resultSet.insertId: ' + resultSet.insertId);
    console.log('resultSet.rowsAffected: ' + resultSet.rowsAffected);
  }, function(tx, error) {
    console.log('INSERT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

In case of a read-only transaction, it is possible to use `readTransaction` which will not use BEGIN, COMMIT, or ROLLBACK:

```Javascript
db.readTransaction(function(tx) {
  tx.executeSql("SELECT UPPER('Some US-ASCII text') AS uppertext", [], function(tx, resultSet) {
    console.log("resultSet.rows.item(0).uppertext: " + resultSet.rows.item(0).uppertext);
  }, function(tx, error) {
    console.log('SELECT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

**WARNING:** It is NOT allowed to execute sql statements on a transaction after it has finished. Here is an example from the **Populating Cordova SQLite storage with the JQuery API post** at <http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html>:

```Javascript
  // BROKEN SAMPLE:
  var db = window.sqlitePlugin.openDatabase({name: "test.db"});
  db.executeSql("DROP TABLE IF EXISTS tt");
  db.executeSql("CREATE TABLE tt (data)");

  db.transaction(function(tx) {
    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        $.each(res, function(i, item) {
          console.log('REPO NAME: ' + item.name);
          tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item.name));
        });
      }
    });
  }, function(e) {
    console.log('Transaction error: ' + e.message);
  }, function() {
    // Check results:
    db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
      console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
    });
  });
```

You can find more details and a step-by-step description how to do this right in the **Populating Cordova SQLite storage with the JQuery API** post at: <http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html>

**NOTE/minor bug:** Just like the single-statement transaction described above, the object returned by `resultSet.rows.item(rowNumber)` is **not** immutable. In addition, multiple calls to `resultSet.rows.item(rowNumber)` with the same `rowNumber` on the same `resultSet` object return the same object. For example, the following code will show `Second uppertext result: ANOTHER`:

```Javascript
db.readTransaction(function(tx) {
  tx.executeSql("SELECT UPPER('First') AS uppertext", [], function(tx, resultSet) {
    var obj1 = resultSet.rows.item(0);
    obj1.uppertext = 'ANOTHER';
    console.log('Second uppertext result: ' + resultSet.rows.item(0).uppertext);
    console.log('SELECT error: ' + error.message);
  });
});
```

**FUTURE TBD:** It should be possible to get a row result object using `resultSet.rows[rowNumber]`, also in case of a single-statement transaction. This is non-standard but is supported by the Chrome desktop browser.

<!-- END Standard asynchronous transactions -->

## Background processing

The threading model depends on which version is used:
- For Android, one background thread per db;
- for iOS/macOS, background processing using a very limited thread pool (only one thread working at a time);
- for Windows, no background processing.

<!-- END Background processing -->

## Sample with PRAGMA feature

Creates a table, adds a single entry, then queries the count to check if the item was inserted as expected. Note that a new transaction is created in the middle of the first callback.

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default'});

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    // demonstrate PRAGMA:
    db.executeSql("pragma table_info (test_table);", [], function(res) {
      console.log("PRAGMA res: " + JSON.stringify(res));
    });

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      db.transaction(function(tx) {
        tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
          console.log("res.rows.length: " + res.rows.length + " -- should be 1");
          console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
        });
      });

    }, function(e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

**NOTE:** PRAGMA statements must be executed in `executeSql()` on the database object (i.e. `db.executeSql()`) and NOT within a transaction.

<!-- END Sample with PRAGMA feature -->

## Sample with transaction-level nesting

In this case, the same transaction in the first executeSql() callback is being reused to run executeSql() again.

```js
// Wait for Cordova to load
document.addEventListener('deviceready', onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: 'my.db', location: 'default'});

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
        console.log("res.rows.length: " + res.rows.length + " -- should be 1");
        console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
      });

    }, function(tx, e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

This case will also works with Safari (WebKit), assuming you replace `window.sqlitePlugin.openDatabase` with `window.openDatabase`.

<!-- END Sample with transaction-level nesting -->

## Close a database object

This will invalidate **all** handle access handle objects for the database that is closed:

```js
db.close(successcb, errorcb);
```

It is OK to close the database within a transaction callback but *NOT* within a statement callback. The following example is OK:

```Javascript
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
  });
}, function(error) {
  // OK to close here:
  console.log('transaction error: ' + error.message);
  db.close();
}, function() {
  // OK to close here:
  console.log('transaction ok');
  db.close(function() {
    console.log('database is closed ok');
  });
});
```

The following example is NOT OK:

```Javascript
// BROKEN:
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
    // BROKEN - this will trigger the error callback:
    db.close(function() {
      console.log('database is closed ok');
    }, function(error) {
      console.log('ERROR closing database');
    });
  });
});
```

**BUG:** It is currently NOT possible to close a database in a `db.executeSql` callback. For example:

```Javascript
// BROKEN DUE TO BUG:
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (res) {
  var stringlength = res.rows.item(0).stringlength;
  console.log('got stringlength: ' + res.rows.item(0).stringlength);

  // BROKEN - this will trigger the error callback DUE TO BUG:
  db.close(function() {
    console.log('database is closed ok');
  }, function(error) {
    console.log('ERROR closing database');
  });
});
```

**SECOND BUG:** When a database connection is closed, any queued transactions are left hanging. All pending transactions should be errored when a database connection is closed.

**NOTE:** As described above, if multiple database access handle objects are opened for the same database and one database handle access object is closed, the database is no longer available for the other database handle objects. Possible workarounds:
- It is still possible to open one or more new database handle objects on a database that has been closed.
- It *should* be OK not to explicitly close a database handle since database transactions are [ACID](https://en.wikipedia.org/wiki/ACID) compliant and the app's memory resources are cleaned up by the system upon termination.

**FUTURE TBD:** `dispose` method on the database access handle object, such that a database is closed once **all** access handle objects are disposed.

<!-- END Close a database object -->

## Delete a database

```js
window.sqlitePlugin.deleteDatabase({name: 'my.db', location: 'default'}, successcb, errorcb);
```

with `location` or `iosDatabaseLocation` parameter *required* as described above for `openDatabase` (affects iOS/macOS *only*)

**BUG:** When a database is deleted, any queued transactions for that database are left hanging. All pending transactions should be errored when a database is deleted.

<!-- END Delete a database -->

## Database schema versions

The transactional nature of the API makes it relatively straightforward to manage a database schema that may be upgraded over time (adding new columns or new tables, for example). Here is the recommended procedure to follow upon app startup:
- Check your database schema version number (you can use `db.executeSql` since it should be a very simple query)
- If your database needs to be upgraded, do the following *within a single transaction* to be failure-safe:
  - Create your database schema version table (single row single column) if it does not exist (you can check the `sqlite_master` table as described at: http://stackoverflow.com/questions/1601151/how-do-i-check-in-sqlite-whether-a-table-exists)
  - Add any missing columns and tables, and apply any other changes necessary

**IMPORTANT:** Since we cannot be certain when the users will actually update their apps, old schema versions will have to be supported for a very long time.

<!-- END Database schema versions -->

## Use with Ionic/ngCordova/Angular

It is recommended to follow the tutorial (*with some adaptations*) at: https://blog.nraboy.com/2014/11/use-sqlite-instead-local-storage-ionic-framework/

A sample is provided at: [litehelpers / Ionic-sqlite-database-example](https://github.com/litehelpers/Ionic-sqlite-database-example)

Documentation at: <http://ngcordova.com/docs/plugins/sqlite/>

Other resource(s):
- <https://www.packtpub.com/books/content/how-use-sqlite-ionic-store-data>

<!-- END Use with Ionic/ngCordova/Angular -->

# Installing

## Easy installation with Cordova CLI tool

```shell
npm install -g cordova # (in case you don't have cordova)
cordova create MyProjectFolder com.my.project MyProject && cd MyProjectFolder # if you are just starting
cordova plugin add https://github.com/litehelpers/Cordova-sqlite-evcore-common-free --save
cordova platform add <desired platform> # repeat for all desired platform(s)
cordova prepare # OPTIONAL (MANDATORY cordova-ios older than 4.3.0 (Cordova CLI 6.4.0))
```

**Additional Cordova CLI NOTES:**

- It is recommended to add *all* plugins including standard plugins such as `cordova-plugin-whitelist` with the `--save` flag to track these in `config.xml`.
- In general there is no need to keep the Cordova `platforms` subdirectory tree in source code control (such as git). In case *all* plugins are added with the `--save` flag then there is no need to keep the `plugins` subdirectory tree in source code control either.
- It is **MANDATORY** to use `cordova prepare` in case of cordova-ios older than 4.3.0 (Cordova CLI 6.4.0).
- In case of problems with building and running it is recommended to try again after `cordova prepare`.
- If you cannot build for a platform after `cordova prepare`, you may have to remove the platform and add it again, such as:

```shell
cordova platform rm ios
cordova platform add ios
```

or more drastically:

```shell
rm -rf platforms
cordova platform add ios
```

You can find some more details in a nice writeup (though with old links and package names): <http://iphonedevlog.wordpress.com/2014/04/07/installing-chris-brodys-sqlite-database-with-cordova-cli-android/>.

<!-- END Easy installation with Cordova CLI tool -->

## Plugin installation sources

- https://github.com/litehelpers/Cordova-sqlite-evcore-common-free - latest version

<!-- END Plugin installation sources -->

## Windows platform usage

This plugin can be challenging to use on Windows since it includes a native SQLite3 library that is built as a part of the Cordova app. Here are some requirements:

- It is **not** possible to use this plugin with the default "Any CPU" target. A specific CPU target must be selected.
- It is **not** allowed to manually change the app id in the Windows platform. The Windows app ID may be declared:
  - using a `windows-identity-name` attribute (ref: <http://phonegap.com/blog/2016/04/25/windows-10-and-phonegap-cli-6_1-now-on-build/>);
  - "WindowsStoreIdentityName" setting (ref: <https://cordova.apache.org/docs/en/latest/config_ref/>).

<!-- END Windows platform usage -->

## Installation test

### Easy installation test

Use `window.sqlitePlugin.echoTest` and/or `window.sqlitePlugin.selfTest` as described above (please wait for the `deviceready` event).

### Quick installation test

Assuming your app has a recent template as used by the Cordova create script, add the following code to the `onDeviceReady` function, after `app.receivedEvent('deviceready');`:

```Javascript
  window.sqlitePlugin.openDatabase({ name: 'hello-world.db', location: 'default' }, function (db) {
    db.executeSql("select length('tenletters') as stringlength", [], function (res) {
      var stringlength = res.rows.item(0).stringlength;
      console.log('got stringlength: ' + stringlength);
      document.getElementById('deviceready').querySelector('.received').innerHTML = 'stringlength: ' + stringlength;
   });
  });
```

<!-- END Installation test -->

# Support

## Free support policy

Free support is provided on a best-effort basis and is only available in public forums. Please follow the steps below to be sure you have done your best before requesting help. Use with Ionic and other Angular derivatives is no longer covered by free support.

## Professional support

Professional support is available by contacting: <sales@litehelpers.net>

For more information: <http://litehelpers.net/>

## Before seeking help

First steps:
- Verify that you have followed the steps in [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist)
- Try the self-test functions as described above
- Check the pitfalls and troubleshooting steps in the [pitfalls](#pitfalls) section and [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls) for possible troubleshooting

and check the following:
- You are using the latest version of the Plugin (Javascript and platform-specific part) from this repository.
- The plugin is installed correctly.
- You have included the correct version of `cordova.js`.
- You have registered the plugin properly in `config.xml`.

If you still cannot get something to work:
- create a fresh, clean Cordova project, ideally based on [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test);
- add this plugin according to the instructions above;
- double-check that you follwed the steps in [brodybits / Cordova-quick-start-checklist](https://github.com/brodybits/Cordova-quick-start-checklist);
- try a simple test program;
- double-check the pitfalls in [brodybits / Avoiding-some-Cordova-pitfalls](https://github.com/brodybits/Avoiding-some-Cordova-pitfalls)

## Issues with AJAX

General: As documented above with a negative example the application must wait for the AJAX query to finish before starting a transaction and adding the data elements.

In case of issues *it is recommended to* rework the reproduction program insert the data from a JavaScript object after a delay. There is already a test function for this in [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test).

FUTURE TBD examples

## Test program to seek help

If you continue to see the issue: please make the simplest test program possible based on [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) to demonstrate the issue with the following characteristics:
  - it completely self-contained, i.e. it is using no extra libraries beyond cordova & SQLitePlugin.js;
  - if the issue is with *adding* data to a table, that the test program includes the statements you used to open the database and create the table;
  - if the issue is with *retrieving* data from a table, that the test program includes the statements you used to open the database, create the table, and enter the data you are trying to retrieve.

## What will be supported for free

It is *recommended* to make a small, self-contained test program based on [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test) that can demonstrate your problem and post it. Please do not use any other plugins or frameworks than are absolutely necessary to demonstrate your problem.

In case of a problem with a pre-populated database, please post your entire project.

## What is NOT supported for free

- Use with Ionic and other Angular derivatives is no longer covered by free support.

## Support for issues with Angular/"ngCordova"/Ionic

Professional support is available for use with Ionic and other forms of ngCordova/Angular. For more information please contact: <sales@litehelpers.net>

## What information is needed for help

Please include the following:
- Which platform(s) Android/iOS/macOS/Windows 8.1/Windows Phone 8.1/Windows 10
- Clear description of the issue
- A small, complete, self-contained program that demonstrates the problem, preferably as a Github project, based on [brodybits / Cordova-sqlite-bootstrap-test](https://github.com/brodybits/Cordova-sqlite-bootstrap-test). ZIP/TGZ/BZ2 archive available from a public link is OK. No RAR or other such formats please!
- In case of a Windows build problem please capture the entire compiler output.

## Please do NOT use any of these formats

- screen casts or videos
- RAR or similar archive formats
- Intel, MS IDE, or similar project formats unless absolutely necessary

## Where to ask for help

Once you have followed the directions above, you may request free support in the following location(s):
- [litehelpers / Cordova-sqlite-evcore-common-free / issues](https://github.com/litehelpers/Cordova-sqlite-evcore-common-free/issues)
- [litehelpers / Cordova-sqlite-help](https://github.com/litehelpers/Cordova-sqlite-help)

Please include the information described above otherwise.

<!-- END Support -->

# Unit tests

Unit testing is done in `spec`.

## running tests from shell

To run the tests from \*nix shell, simply do either:

    ./bin/test.sh ios

or for Android:

    ./bin/test.sh android

To run from a windows powershell (here is a sample for android target):

    .\bin\test.ps1 android

<!-- END Unit tests -->

# Adapters

## Lawnchair Adapter

### Common adapter

Please look at the `Lawnchair-adapter` tree that contains a common adapter, which should also work with the Android version, along with a test-www directory.

### Included files

Include the following Javascript files in your HTML:

- `cordova.js` (don't forget!)
- `lawnchair.js` (you provide)
- `SQLitePlugin.js` (in case of Cordova pre-3.0)
- `Lawnchair-sqlitePlugin.js` (must come after `SQLitePlugin.js` in case of Cordova pre-3.0)

### Lawnchair Sample

The `name` option determines the sqlite database filename, *with no extension automatically added*. Optionally, you can change the db filename using the `db` option.

In this example, you would be using/creating a database with filename `kvstore`:

```Javascript
kvstore = new Lawnchair({name: "kvstore"}, function() {
  // do stuff
);
```

Using the `db` option you can specify the filename with the desired extension and be able to create multiple stores in the same database file. (There will be one table per store.)

```Javascript
recipes = new Lawnchair({db: "cookbook", name: "recipes", ...}, myCallback());
ingredients = new Lawnchair({db: "cookbook", name: "ingredients", ...}, myCallback());
```

**KNOWN ISSUE:** Not all db options are supported by the Lawnchair adapter. The workaround is to first open the database file using `sqlitePlugin.openDatabase()`.

### PouchDB

- [nolanlawson / pouchdb-adapter-cordova-sqlite](https://github.com/nolanlawson/pouchdb-adapter-cordova-sqlite)

## Adapters to be supported in the future

- IndexedDBShim

<!-- END Adapters -->

# Sample

Contributed by [@Mikejo5000 (Mike Jones)](https://github.com/Mikejo5000) from Microsoft.

## Interact with the SQLite database

The SQLite storage plugin sample allows you to execute SQL statements to interact with the database. The code snippets in this section demonstrate simple plugin tasks including:

* [Open the database and create a table](#openDb)
* [Add a row (record) to the database](#addRow)
* [Read rows from the database](#readRow) that match a column value
* [Remove a row from the database](#removeRow) that matches a column value
* [Update rows in the database](#updateRow) that match a column value
* [Close the database](#closeDb)

##Open the database and create a table <a name="openDb"></a>

Call the `openDatabase()` function to get started, passing in the name and location for the database.

```Javascript
var db = window.sqlitePlugin.openDatabase({ name: 'my.db', location: 'default' }, function (db) {

    // Here, you might create or open the table.

}, function (error) {
    console.log('Open database ERROR: ' + JSON.stringify(error));
});
```

Create a table with three columns for first name, last name, and a customer account number. If the table already exists, this SQL statement opens the table.

```Javascript
db.transaction(function (tx) {
    // ...
    tx.executeSql('CREATE TABLE customerAccounts (firstname, lastname, acctNo)');
}, function (error) {
    console.log('transaction error: ' + error.message);
}, function () {
    console.log('transaction ok');
});
```

By wrapping the previous `executeSql()` function call in `db.transaction()`, we will make these tasks asynchronous. If you want to, you can use multiple `executeSql()` statements within a single transaction (not shown).

## Add a row to the database <a name="addRow"></a>

Add a row to the database using the INSERT INTO SQL statement.

```Javascript
function addItem(first, last, acctNum) {

    db.transaction(function (tx) {

        var query = "INSERT INTO customerAccounts (firstname, lastname, acctNo) VALUES (?,?,?)";

        tx.executeSql(query, [first, last, acctNum], function(tx, res) {
            console.log("insertId: " + res.insertId + " -- probably 1");
            console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
        },
        function(tx, error) {
            console.log('INSERT error: ' + error.message);
        });
    }, function(error) {
        console.log('transaction error: ' + error.message);
    }, function() {
        console.log('transaction ok');
    });
}
```

To add some actual rows in your app, call the `addItem` function several times.

```Javascript
addItem("Fred", "Smith", 100);
addItem("Bob", "Yerunkle", 101);
addItem("Joe", "Auzomme", 102);
addItem("Pete", "Smith", 103);
```

##Read data from the database <a name="readRow"></a>

Add code to read from the database using a SELECT statement. Include a WHERE condition to match the resultSet to the passed in last name.

```Javascript
function getData(last) {

    db.transaction(function (tx) {

        var query = "SELECT firstname, lastname, acctNo FROM customerAccounts WHERE lastname = ?";

        tx.executeSql(query, [last], function (tx, resultSet) {

            for(var x = 0; x < resultSet.rows.length; x++) {
                console.log("First name: " + resultSet.rows.item(x).firstname +
                    ", Acct: " + resultSet.rows.item(x).acctNo);
            }
        },
        function (tx, error) {
            console.log('SELECT error: ' + error.message);
        });
    }, function (error) {
        console.log('transaction error: ' + error.message);
    }, function () {
        console.log('transaction ok');
    });
}
```

##Remove a row from the database <a name="removeRow"></a>

Add a function to remove a row from the database that matches the passed in customer account number.

```Javascript
function removeItem(acctNum) {

    db.transaction(function (tx) {

        var query = "DELETE FROM customerAccounts WHERE acctNo = ?";

        tx.executeSql(query, [acctNum], function (tx, res) {
            console.log("removeId: " + res.insertId);
            console.log("rowsAffected: " + res.rowsAffected);
        },
        function (tx, error) {
            console.log('DELETE error: ' + error.message);
        });
    }, function (error) {
        console.log('transaction error: ' + error.message);
    }, function () {
        console.log('transaction ok');
    });
}
```

##Update rows in the database <a name="updateRow"></a>

Add a function to update rows in the database for records that match the passed in customer account number. In this form, the statement will update multiple rows if the account numbers are not unique.

```Javascript
function updateItem(first, id) {
    // UPDATE Cars SET Name='Skoda Octavia' WHERE Id=3;
    db.transaction(function (tx) {

        var query = "UPDATE customerAccounts SET firstname = ? WHERE acctNo = ?";

        tx.executeSql(query, [first, id], function(tx, res) {
            console.log("insertId: " + res.insertId);
            console.log("rowsAffected: " + res.rowsAffected);
        },
        function(tx, error) {
            console.log('UPDATE error: ' + error.message);
        });
    }, function(error) {
        console.log('transaction error: ' + error.message);
    }, function() {
        console.log('transaction ok');
    });
}
```

To call the preceding function, add code like this in your app.

```Javascript
updateItem("Yme", 102);
```

##Close the database <a name="closeDb"></a>

When you are finished with your transactions, close the database. Call `closeDB` within the transaction success or failure callbacks (rather than the callbacks for `executeSql()`).

```Javascript
function closeDB() {
    db.close(function () {
        console.log("DB closed!");
    }, function (error) {
        console.log("Error closing DB:" + error.message);
    });
}
```

<!-- END Sample -->

## Source tree

- `SQLitePlugin.coffee.md`: platform-independent (Literate coffee-script, can be read by recent coffee-script compiler)
- `www`: platform-independent Javascript as generated from `SQLitePlugin.coffee.md` (and committed!)
- `src`: platform-specific source code
- `node_modules`: placeholder for external dependencies
- `scripts`: installation hook script to fetch the external dependencies via `npm`
- `spec`: test suite using Jasmine (`2.4.1`)
- `tests`: very simple Jasmine test suite that is run on Circle CI (Android version) and Travis CI (iOS version) (used as a placeholder)
- `Lawnchair-adapter`: Lawnchair adaptor, based on the version from the Lawnchair repository, with the basic Lawnchair test suite in `test-www` subdirectory

<!-- END Source tree -->

# Contributing

## Community

- Testimonials of apps that are using this plugin would be especially helpful.
- Reporting issues can help improve the quality of this plugin.

**NOTE:** As stated above, patches will *NOT* be accepted on this project due to potential licensing issues. Issues with reproduction scenarios will help maintain and improve the quality of this plugin for future users. (It is also helpful if you have a pointer to what code may be causing the issue.)

<!-- END Contributing -->

## Contact

<sales@litehelpers.net>
