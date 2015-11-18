# JSBandwidth

To test inside a browser the bandwidth, there's no easy way. This is what JsBandwidth tries to achieve.

This project was initially forked from https://code.google.com/p/jsbandwidth/.

## License
I decided to keep the same license as the initial project, [MIT](http://opensource.org/licenses/mit-license.php).

## Server-side set-up
1. Set up a web server of your choice.
2. Depending on your web server, drop the corresponding project files in your web server's document root (or a sub-directory, if you wish). What `src/main/webapp/post.*` file to choose depends on your web server. The upload test needs to be able to send a POST request to the server. The receiving page doesn't have to do anything with the data. However, some servers will not allow you to send a POST request to a .htm file. Therefore, the project includes several blank server side script files (post.aspx, post.php, post.pl). `src/main/webapp/test.bin` is mandatory, but it's nothing more than random bytes. 

### Spring Controller

If you want to use a Spring Controller to post test data you can define a controller method like this

	@RequestMapping("/test-post")
	public @ResponseBody String testPost() {
		return "true";
	}
	
and then specify `options.uploadUrl='/test-post'`.

Please be aware that some servers, like Tomcat, by their default setup can impose a limit on the upload data size to avoid DoS attacks. You either modify that setup or specify `options.uploadDataMaxSize`.

## Testing
To run the tests you need to

1. Install a web server, e.g. `http-server`: `npm install -g http-server`
2. Start the web server in the project folder: `http-server -p 8081 .`
3. Access from any browser: [http://localhost:8081/src/test/test.html](http://localhost:8081/spec/JsBandwidthSpec.html)

## Installation
    
    npm install jsbandwidth

## JavaScript API
The JavaScript API works with both Angular and jQuery, depending on what library is included (if both, Angular is preferred).

First you need to get hold of the `jsBandwidth` object.

- In Angular

<pre><code>
myApp.controller('JsBandwidthTestController', ["$scope", "jsBandwidth", function ($scope, jsBandwidth) {
	$scope.test = function(options, callback) {
		jsBandwidth.testSpeed(options)
				.then(function(result) {
						$scope.downloadSpeed = result.downloadSpeed;
						$scope.uploadSpeed = result.uploadSpeed;
						$scope.errorStatus = null;
						callback();
					}
					, function(error) {
						$scope.downloadSpeed = -1;
						$scope.uploadSpeed = -1;
						$scope.errorStatus = error.status;
						callback();
					});
	};
}]);
</code></pre>

- In jQuery

<pre><code>
	var jsBandwidth = $.jsBandwidth;
</code></pre>

or 

<pre><code>
	var jsBandwidth = jQuery.jsBandwidth;
</code></pre>

- With require

<pre><code>
	var jsBandwidth = require("jsbandwidth");
</code></pre>

The `jsBandwidth` object has 3 methods with a similar signature:
- `testLatency(options)`
- `testDownloadSpeed(options)`
- `testUploadSpeed(options)`
- `testSpeed(options)` which combines all the above into one

The `options` parameter is an object and it has the following fields
- `latencyTestUrl` the URL used for latency testing. Usually a big binary content is expected to be downloaded.
- `downloadUrl` the URL used for download speed testing. Usually a big binary content is expected to be downloaded.
- `uploadUrl` the URL used for upload speed testing. It should accept a POST method.
- `uploadData` the data that is sent to the server to test the upload
- `uploadDataMaxSize` if specified `uploadData` is going to be truncated to this maximum length. Some servers, like Tomcat, by their default setup can impose a limit on the upload data size to avoid DoS attacks. You either modify that setting or use `options.uploadDataMaxSize`. The usual limit is 2Mb.
- `uploadDataSize` if `uploadData` is not specified, then a chunk of this size is randomly generated instead
- 'ajax' the AJAX service, either from jQuery or $http from Angular. If not specified, it will be automatically detected depending whether jQuery or Angular is included.

All three methods return a [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) and you can use the `then` method. That promise is also augmented with a `cancel()` method.

### Example

	var jsBandwidth = require("jsbandwidth");
	jsBandwidth.testSpeed(options)
		.then(function (result) {
				console.log("Latency is " + result.latency + " ms, download speed is " + result.downloadSpeed + "bps and upload speed is " result.uploadSpeed + "bps");
			},
			function(error) {
				console.log("An error occured during net speed test.");
			});

### Angular controller			
An Angular controller, called `JsBandwidthController`, is provided for your convenience. The controller uses the service and it defines the following fields/methods in the scope
- `test` this is the service running the speed test. If null or undefined, there's no test currently running, so it can be used for checking if a speed test is currently running.
- `options` the options used to run the speed test
- 'result.latency' the estimated latency in ms. If `result` is null or undefined, the test is in progress or ended with an error.
- `result.downloadSpeed` the estimated download speed in bps.
- `result.uploadSpeed` the estimated upload speed in bps.
- `error` if null or undefined, then a test is in progress or completed successfully. If not null, then an error occured during the last speed test.
-  `error.status` the error status

'complete` event is emitted when the test is completed or 'error' if an error occured.

Below is an example on how to use it in your page:

	<div data-ng-controller="JsBandwidthController" class="netSpeedTest"
			data-ng-init="options.downloadUrl='/test.bin';">
		<span data-ng-if="error">
			<span>Error</span>: <span data-ng-bind="error.status"></span>
		</span>
		<span data-ng-if="result">
			<span>Latency:</span>
			<span data-ng-bind="convertToMbps(result.latency)"></span><span th:text="#{Mbps}"></span>
			<span>Download speed:</span>
			<span data-ng-bind="convertToMbps(result.downloadSpeed)"></span><span th:text="#{Mbps}"></span>
			<span>Upload speed</span>
			<span data-ng-bind="convertToMbps(result.uploadSpeed)"></span><span th:text="#{Mbps}"></span>
		</span>
		<button data-ng-if="!test" data-ng-click="start()" class="start">Start test</button>
		<button data-ng-if="test" data-ng-click="cancel()" class="cancel">Cancel test</button>
	</div>


### Formatting
The speed is calculated in bps (bits per second). In the Angular controller you have the method `convertToMbps` for your convenience. If you want to format it differently, you can use [js-quantities](https://github.com/gentooboontoo/js-quantities).
