# jquery-data-remote

jquery-data-remote is a plugin that simplifies the common task of making api/remote requests and injecting the response to the page. Optionally leverages Handlebars.js for templating. Inspired by ruby on rails' unobtrusive scripting adapter for jQuery (https://github.com/rails/jquery-ujs).

[![Travis build status](https://api.travis-ci.org/codfish/jquery-data-remote.svg?branch=master)](https://travis-ci.org/codfish/jquery-data-remote)

## Install

* [yarn](https://yarnpkg.com/en/package/jquery-data-remote): `yarn add jquery-data-remote`
* [npm](http://npmjs.org/package/jquery-data-remote): `npm install --save jquery-data-remote`
* [cdnjs](https://cdnjs.com/libraries/jquery-data-remote)
* [Download the latest release on Github](https://github.com/codfish/jquery-data-remote/releases)

## Usage

Here's a great example of how this plugin can help minimize the amount of work you need to do in order to make an ajax request and handle the results. Here's an api request to githubs' gists api, using handlebars for templating (optional). This also leverages all of the default options (event type of 'load', data type of json, request type of GET, etc. See [Options](#options) for more details).

  ```html
  <div data-url="https://api.github.com/users/codfish/gists">
    <script type="text/x-handlebars-template">
      <ul>
        {{#each this}}
          <li>
            <h3><a href="{{ html_url }}" target="_blank">{{ html_url }}</a></h3>
            <p>{{ description }}</p>
          </li>
        {{/each}}
      </ul>
    </script>
  </div>
  ```

  ```js
  $('div').dataRemote()
  ```

Almost all options can be set via html5 data attributes, or passed into the plugin initialization, or a mixture of both. Note html5 data attributes take precedence, so if you wanted to, you can initialize data remote with options, and then override options on any specific element using data attributes. See the following example:

Here's an example with 3 elements getting initialized together, but showing the ability to override options with data attributes.

  - Here's the first element, a wrapper element with no data attributes. Leverages the default options of the plugin, as well as the options you specify below in the plugin call.

  ```html
  <ul class="get-news">
    <script type="text/x-handlebars-template">
      {{#each news_items}}
        <li>
          <h3><a href="{{ url }}" target="_blank">{{ url }}</a></h3>
          <p>{{ dek }}</p>
        </li>
      {{/each}}
    </script>
  </ul>
  ```

  ```js
  $('.get-news').dataRemote({
    url: 'http://api.example.com/news',
  });
  ```

  - Now we add a link to the page to allow the user to load more news. This will **override** the request url specified in the options object we passed in when we initialized the plugin.

  ```html
  <a class="get-news"
     data-target=".news-list-wrapper"
     data-event-type="click"
     data-placement="append"
     data-url="http://api.example.com/news?page=2">Load More News</a>
  ```

## Options

#### url {string} (**Required**)

API Request URL. Can be absolute or relative. Cross browser requests obviously adhere to CORS. For cross browser requests, you must either set the `dataType` option to `jsonp` or the API request must be to a public api/endpoint.

```html
<div data-remote="true" data-url="https://api.github.com/users/codfish/gists"></div>
<script>
$('[data-remote="true"]').dataRemote();
</script>
```

```html
<div class="news-list"></div>
<script>
$(".news-list").dataRemote({
  url: ''
  template: '#news-item-template',
});
</script>
```

#### data {Object} (default: `{}`)

Request data. If this is passed as a html5 attribute, it needs to be a valid JSON string.

```html
<div class="news-list"></div>
<script>
$(".news-list").dataRemote({
  data: {
    page: 1,
    count: 20,
    type: 'videos'
  }
});
</script>
```

*OR*

```html
<div class="news-list" data-data='{"page": 2, "count": 20, "type": "videos"}'></div>
<script>
$(".news-list").dataRemote();
</script>
```

#### dataType {string} (default: `'json'`)

The type of response data you're expecting. Can be any dataType value supported by [jQuery.ajax](http://api.jquery.com/jquery.ajax/).

#### debug {boolean} (default: `false`)

Turn debugging on/off. Will output errors/notices to the console.

#### eventType {string} (default: `'load'`)

The event type to fire data request on. Can be any event type that is supported by jQuery, including custom events.

#### handlebars {boolean} (default: `false`)

Whether to use handlebars templating engine. If you put a handlebars template within the target element, Handlebars will be used, regardless of this options' value.

#### loaderImg {string} (default: `null`)

Source of an optional loader image. When you want a loader image to appear in the target element, while the ajax request is being made. You are responsible for styling how you want, however.

```html
<script>
$(".news-list").dataRemote({
  loaderImg: '/images/loader.gif'
});
</script>
```

#### method {string} (default: `GET`)

The HTTP method to use for the request (e.g. "POST", "GET", "PUT") Can be any request type supported by jQuery.

#### oneAndDone {boolean} (default: `true`)

Whether to remove the event binding after the initial request.

#### placement {string} (default: `html`)

Where to inject response relative to target (uses jQuery DOM insertion methods. Can be 'html', 'append', 'prepend', 'before' or 'after').

#### target {string} (default: `$(element).selector`)

Selector of the element where you want your response injected. By default it's assumed that the target is the element that data remote has been initialized on.

#### template {string} (default: `''`)

Selector of the handlebars template. By default it will look inside target element for the template.

#### type {string} (default: `'GET'`)

Alias for method.

#### debounceEvents {Array} (default: `['keyup', 'keydown', 'keypress', 'scroll', 'resize']`)

A list of event types that will cause the execution of the main callback function to be [debounced](http://davidwalsh.name/javascript-debounce-function). The default events provided are one's that can typically get fired in rapid succession and cause jank. You cannot provide this option as a data attribute. Custom events can be provided.

```html
<script>
$(".news-list").dataRemote({
  debounceEvents: ['scroll', 'gesturechange', 'orientationchange', 'customresize']
});
</script>
```


## Callbacks

#### before {Function} (default: `function($target) {}`)

Before callback. Fires directly before the request is made. Default is an empty function.

#### complete {Function} (default: `function($target) {}`)

Complete callback. Fires after request, on success or error. Default is an empty function.

#### error {Function} (default: `errorCallback($target, options, response, error)`)

Error callback. Fires if the ajax request fails. Takes 4 arguments. Default error callback (`errorCallback()`) handles error reporting is `debug` is true.

#### success {Function} (default: `successCallback($target, options, response)`)

Success callback. Fires on the success of the ajax request. Takes 3 arguments. The default success callback (`successCallback()`) handles templating the response.

## Demo

To view the demo, run the following:

```sh
gulp demo
```

## In Development

- [x] ~~Add a node server via gulp, so you can use that to serve the demo, with a simple `gulp serve`~~
- [ ] Support for multiple templating engines, https://github.com/codfish/jquery-data-remote/issues/5
- [ ] Add better support for POST requests
- [ ] Add support for authenticated requests
- [ ] Add better error handling
