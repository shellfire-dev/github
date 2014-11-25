# `github`: functions module for [shellfire]

This module provides a simple framework for using [GitHub v3 REST API](https://developer.github.com/v3/) with a [shellfire] application. It currently supports a subset of features for managing releases. We welcome contributions to increase the coverage.

## Overview

To use the API, you first need to initialise it. We use OAuth2 Personal Access Tokens, which you can create yourself.

```bash
github_api_v3_initialise "$tokenFilePath"
```

`tokenFilePath` should be a path to a file that contains one line with an OAuth2 Personal Access Token in it. You can create these tokens using the settings for your account. On no account should you check in this file! You may want to add a rule to your `.gitignore` file just to make sure.

You can then make API calls. For example, to retrieve a list of release URLs:-

```bash
github_api_v3_releases_list "$owner" "$repo"

url=''
some_event_callback()
{
	if jsonreader_eventMatches ":${jsonreader_path_index}/url" string; then
		echo "Url for release is $eventValue"
	fi
}
jsonreader_parse "$github_api_v3_responseFilePath" some_event_callback
```

This example uses the [jsonreader] parser and event handler. For an example of how to handle more complex needs, see the function `_github_api_v3_errorMessage_event()` in the [v3.functions source code](https://github.com/shellfire-dev/github/blob/master/api/v3/v3.functions).


## Importing

To import this module, add a git submodule to your repository. From the root of your git repository in the terminal, type:-
```bash
mkdir -p lib/shellfire
cd lib/shellfire
git submodule add "https://github.com/shellfire-dev/github.git"
cd -
git submodule init --update
```

You may need to change the url `https://github.com/shellfire-dev/github.git` above if using a fork.

You will also need to add paths - include the module [paths.d].

You will also need to import the [curl], [urlencode], [jsonreader], [jsonwriter], [unicode] and [version] modules.


## Namespace `github_api_v3`

This namespace contains the core functionality needed to access the API.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn github/api v3
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn github/api v3
	…
}
```

### Functions

***
#### `github_api_v3_initialise`

|Parameter|Value|Optional|
|---------|-----|--------|
|`tokenFilePath`|Path to a file that contains one line with an OAuth2 Personal Access Token in it.|_No_|
|`github_api_v3_endpoint`|Base URL of endpoint (for GitHub Enterprise). Defaults to `https://api.github.com`.|_Yes_|

Creates temporary file paths for request and response bodies in `github_api_v3_responseFilePath` and `github_api_v3_requestFilePath` (the latter can be overriden by setting `curl_uploadFile`).

***
#### `github_api_v3_timestampToEpochSeconds`

|Parameter|Value|Optional|
|---------|-----|--------|
|`timestamp`|Timestamp from GitHub|_No_|

Helper function to convert GitHub timestamps into epoch seconds (ie seconds since the Unix epoch of 1970).


## Namespace `github_api_v3_releases`

This namespace contains functionality to create releases. It is very closely linked to the underlying REST API, and you should refer to that documentation for specific meanings.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn github/api/v3 releases
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn github/api/v3 releases
	…
}
```

### Functions

***
#### `github_api_v3_releases_list`

|Parameter|Value|Optional|
|---------|-----|--------|
|`owner`|Repository owner|_No_|
|`repo`|Repository|_No_|

Returns a JSON document listing releases in `github_api_v3_responseFilePath`. Use the [jsonreader] module to parse it.

***
#### `github_api_v3_releases_create`

|Parameter|Value|Optional|
|---------|-----|--------|
|`owner`|Repository owner|_No_|
|`repo`|Repository|_No_|
|`tagName`|tag name|_No_|
|`commitish`|commitish|_No_|
|`name`|Name of this release|_No_|
|`body`|A non empty string of markdown|_No_|
|`draft`|`true` or `false`|_No_|
|`prerelease`|`true` or `false`|_No_|

Creates a release.

Sets the variable `github_api_v3_header_Location` to the location URL of the created release.
Sets the variable `github_api_v3_releases_uploadUrlTemplate` to the upload url template for assets.

***
#### `github_api_v3_releases_delete`

|Parameter|Value|Optional|
|---------|-----|--------|
|`owner`|Repository owner|_No_|
|`repo`|Repository|_No_|
|`id`|Release id (eg from `github_api_v3_releases_list()`)|_No_|

Deletes the release.

***
#### `github_api_v3_releases_uploadAsset`

|Parameter|Value|Optional|
|---------|-----|--------|
|`uploadUrlTemplate`|Repository owner|_No_|
|`filePath`|Path to file to upload|_No_|
|`contentType`|MIME Content-Type of file to upload|_No_|
|`label`|Human-friendly file name, effectively|_No_|

Uploads a release asset and then changes its `label` using a `PATCH` only if the `label` doesn't match the basename of `filePath`. `uploadUrlTemplate` can be obtained from `github_api_v3_releases_uploadUrlTemplate` after a call to `github_api_v3_releases_create()`.

[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[jsonreader]: https://github.com/shellfire-dev/jsonreader "jsonreader shellfire module homepage"
[urlencode]: https://github.com/shellfire-dev/urlencode "urlencode shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"
