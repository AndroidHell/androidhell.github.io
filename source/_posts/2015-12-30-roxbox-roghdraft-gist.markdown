---
layout: post
title: "RoxBox roghdraft - gist"
date: 2015-12-30 18:51:38 +0000
comments: true
categories: 
---

/* notes */
Clicks:
Back: exits app (Handled by OS)
Up: Volume Up
Select: Play/Pause
Down: Volume Down

Long-press:
Up: Mute
Down: Skip/Next Track

========================================================================

Prerequisite:

1. Linux Distro (for now)
2. Python
3. Flask
4. Android (for now)
5. Pebble (duh)
6. Pithos (duh)

========================================================================

Follow this guide: http://mrtux.org/projects/commander/docs/quick-start.html

1. Open port: 'sudo ufw allow $serverport'﻿
2. Install python2, pip, git, etc if not already installed (rewrtie if Win/Mac port)
3. Install Flaks: 'sudo pip install flask' or 'sudo pip --user install flask' for global
4. Download and unzip repo, my JSON will be configured. PW is 'roxbox'
5. Run server.py with: 'python server.js'
6. Install watch app. Configure IP to Host machine. set Password to 'roxbox'
7. relaunch watch app after phone app configuration

========================================================================

Coode:

JSON: 
```javascript
{
	"__comment__": "See README.md for a set up guide.",
	"host": "0.0.0.0",
	"port": 5000,
	"debug_mode": true,
	"show_output": true,
	"auth_key": "roxbox",
	"commands":[
		{"title": "Volume Up", "command": "xdotool key XF86AudioRaiseVolume"},
		{"title": "Play/Pause", "command": "xdotool key XF86AudioPlay"},
		{"title": "Volume Down", "command": "xdotool key XF86AudioLowerVolume "},
		{"title": "Mute", "command": "xdotool key XF86AudioMute"},
		{"title": "Skip Track", "command": "xdotool key XF86AudioNext"}
	]
}
```
========================================================================

server.py:
```python
import subprocess, json
from flask import Flask,Response,Markup
app = Flask(__name__)

# Load json file
with open('settings.json') as data_file:
	JSONData = json.load(data_file)

# Use the json data for flask
AUTH_KEY = JSONData["auth_key"]
DEBUG = bool(JSONData["debug_mode"])
SHOW_OUTPUT = bool(JSONData["show_output"])
HTTP_HOST = JSONData["host"]
HTTP_PORT = int(JSONData["port"])

#### FUNCTIONS ####
def listCommandsAsJSON():
	""" Lists all commands in the json file as json """
	count = -1
	for item in JSONData["commands"]:
		count = count + 1
		JSONData["commands"][count]["id"] = count
	
	# Return the json
	return json.dumps(JSONData["commands"], sort_keys=True, indent=4, separators=(',', ': '))


def runCommandFromList(command):
	""" Executes a command from a list/tuple of commands. """

	global SHOW_OUTPUT
	global JSONData

	# We are using try here since we don't want the server to error out if a command id
	# doesn't exist
	try:
		command2run = JSONData["commands"][command]["command"]
	except IndexError:
		# Print an error statement. The code will not continue executing so dont worry
		# about the other lines in the function being executed
		return "invalid command id"

	# Split the command at spaces so it can be passed onto something
	# subprocess.call will accept
	split = command2run.split(' ')
	
	# Execute the command in a way that the output is given to the client not shown in a terminal

	if SHOW_OUTPUT == True:
		return subprocess.check_output(split)
	else:
		subprocess.call(split)

# Settings preview
def getSetting(setting):
	if setting == "debug":
		if DEBUG == True:
			output = "enabled"
		else:
			output = "disabled"
	elif setting == "showoutput":
		if SHOW_OUTPUT == True:
			output = "enabled"
		else:
			output = "disabled"
	else:
			output = "Invalid setting"
	return output

# App Home page
@app.route("/")
def index():
	IndexText = "Commander web server\nSee documentation for information.\n\nSettings:\nDebug mode is " + getSetting('debug')
	IndexText += "\nCommand output is " + getSetting('showoutput')
	return Response(IndexText, mimetype='text/plain')

# Run commands
@app.route("/exec/<authkey>/<int:command_id>")
def commandr(authkey,command_id):
	ResponseText = ""	

	# Show debug information:
	if DEBUG == True:
		ResponseText += "Auth key: " + str(authkey) + "\nCommand-ID: " + str(command_id) + "\n\n"

	# Check if the authkey is correct.
	if authkey == AUTH_KEY:
		# Auth key is correct.
		ResponseText += str(runCommandFromList(command_id))
	else:
		# Auth key isn't correct
		ResponseText += "ERROR: Incorrect authentication key."

	return Response(ResponseText, mimetype='text/plain')

# Page that sends the JSON file so the pebble can read it
@app.route("/send_json/<authkey>")
def send_json(authkey):
	# verify authentication key
	if authkey == AUTH_KEY:
		# auth key is correct, display file.
		#with open("settings.json", "r") as jsonfile:
		#	ResponseText = jsonfile.read()
		ResponseText = listCommandsAsJSON()

	else:
		ResponseText = "ERROR: Incorrect authentication key."
	
	# return response as PLAIN TEXT.
	return Response(ResponseText, mimetype='text/plain')


if __name__ == "__main__":
	app.debug=DEBUG # enable debugging
	app.run(host=HTTP_HOST,port=HTTP_PORT) # run the app
```
========================================================================

app.js
```javascript
var UI = require('ui');
var ajax = require('ajax');
var Settings = require('settings');
var Voice = require("ui/voice");

// get pebble revision
if(Pebble.getActiveWatchInfo) {
	var watchinfo= Pebble.getActiveWatchInfo();
	var platform=watchinfo.platform;
} else {
	platform="aplite";
}

// Set a configurable with the open callback
Settings.config(
	{ url: 'http://androidhell.github.io/settings/' },
	function(e) {
		console.log('opening configurable');
	},
	function(e) {
		console.log('closed configurable');
	}
);


var CONTROL_URL = Settings.option('server'); 
var AUTH_KEY = Settings.option('authkey');
var OUTPUT_MODE = Settings.option('commandoutput');

var APP_VERSION = "1.4";
var ABOUT_TEXT = "Author: Colin Murphy (mrtux@riseup.net)\n\nWebsite: www.mrtux.org/projects/commander";

// Initial window
var initWindow = new UI.Card({
	title: "Pebble Commander",
	scrollable: true,
	style: 'small',
	body: "Getting commands list..."
});
initWindow.show();


if (!CONTROL_URL || !AUTH_KEY) {
	// App not configured
	initWindow.title("App not configured");
	initWindow.body("Enter in the server IP and password on your phone. When done, restart this app.");
	
	if (platform == 'basalt') {
		initWindow.backgroundColor('red');
	}
}
else
{
	// all is good, you can run the app
	runApp();
}

function sendCommand(id, name) {
	// Show a card showing that the command was executed
	var detailCard = new UI.Card({
		backgroundColor: 'white',
		textColor: 'black',
		style: 'small',
		scrollable: true,
		title: name,
		subtitle: "Sending command to server",
		body: ""
	});

	// Show the new Card
	detailCard.show();
	
	ajax({url: 'http://' + CONTROL_URL + "/exec/" + AUTH_KEY + "/" + id, type: 'plain'},
		function (data) {
			detailCard.subtitle("Command sent");
			if (OUTPUT_MODE === true) {
				detailCard.body(data);
			}
			else
			{
				detailCard.body("");
			}
			if (platform == 'basalt') {
				detailCard.backgroundColor('green');
			}

			// hide this after 2 seconds and return to the menu
			setTimeout(function() {detailCard.hide();}, 2000);
		},
		function (error) {
			detailCard.title("Data retrieval failed");
			detailCard.body(error);
			detailCard.subtitle("");
			if (platform == 'basalt') {
				detailCard.backgroundColor('red');
			}
		}
	);
}



function runApp() {
	
	ajax({url: 'http://' + CONTROL_URL + "/send_json/" + AUTH_KEY, type: 'json'},
		function(json) {
			// Data retrieval worked, hide this and show the menu!
			var commandMenu = new UI.Menu({
				backgroundColor: '#AAAAAA',
				textColor: 'black',
				highlightBackgroundColor: 'blue',
				highlightTextColor: 'white',
				fullscreen: false,
				sections: [{
					items: [{"title": "Voice command", "subtitle": "Select then talk"}]	
				},{
					title: 'Commands',
					items: json
				},{
					title: 'Commander',
					items: [
						{"title": "Connection info"},
						{"title": "About", "subtitle": "Version " + APP_VERSION}
					]
				}]
			});
			
			commandMenu.show();
			initWindow.hide();
			
			// Add a click listener for select button click
			commandMenu.on('select', function(event) {
				console.log('Selected item #' + event.itemIndex + ' of section #' + event.sectionIndex);
				console.log('The item is titled "' + event.item.title + '"');
				
				// Item index 0 (list 1) is the voice command option
				if (event.sectionIndex === 0) {
					
					// Dictate voice then run the command if it worked out
					Voice.dictate('start', true, function(e) {
						if (e.err) {
							console.log('Error: ' + e.err);
							return;
						}
						console.log('Transcription: ' + e.transcription);
						
						// search for the name in the list of commands
						for (var i = 0; i < json.length; i++){
							// put the text in lowercase so it can be matched
							var transcription = e.transcription.toLowerCase();
							var itemTitle = json[i].title.toLowerCase();
							
							if (itemTitle == transcription || itemTitle.indexOf(transcription) >= 0) {
								var voiceCommand = json[i].id;
								console.log('Voice command: ' + voiceCommand);
								sendCommand(voiceCommand, json[i].title);
							}
						}
					});

				}
				// If the item is in the 2nd list, its a command
				else if (event.sectionIndex === 1 ) {
					commandMenu.fullscreen(false);
					sendCommand(json[event.itemIndex].id, json[event.itemIndex].title);
				}
				else
				{
					// selection index isn't 0 so this is a different type :)
					
					var infoMenu = new UI.Menu({
						backgroundColor: '#AAAAAA',
						textColor: 'black',
						highlightBackgroundColor: 'blue',
						highlightTextColor: 'white',
					});
					
					if (event.itemIndex === 0) {
						// an id if 0 is the connection settings card
						infoMenu.sections([{
							title: "Connection info",
							items: [
								{"title": "Server", "subtitle": CONTROL_URL},
								{"title": "Auth key", "subtitle": AUTH_KEY}
							]
						}]);
						infoMenu.show();
					}
					else
					{
						// an id of 1 is the about app card
						var aboutCard = new UI.Card({
							style: "small",
							scrollable: true,
							title: ">_ Commander",
							subtitle: "Version " + APP_VERSION,
							body: ABOUT_TEXT
						});
						if (platform == 'basalt') {
							aboutCard.titleColor("#00AA00");
							aboutCard.backgroundColor("#AAAAAA");
							aboutCard.bodyColor("black");
							aboutCard.subtitleColor("#555555");
						}
						aboutCard.show();
					}
				}
			});
		},
		function(error) {
			console.log('Ajax failed: ' + error);
			initWindow.title('Data retrieval failed');
			initWindow.body(error);
			if (platform == 'basalt') {
				initWindow.backgroundColor('red');
				initWindow.textColor('black');
			}
		}
	);
}
```
========================================================================

index.html
```html
<!doctype html>
<html lang="en">
<head>
	<title>RoxBox Settings</title>
	<meta charset="utf-8">
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
	<style>
		.center { text-align: center; }
	</style>
</head>
<body>
<div class="container">
	<h1 class="center page-header">RoxBox Settings</h1>
	
	<form class="form-horizontal">
		<div class="form-group">
			<label for="CommanderServer" class="col-xs-3 control-label">Server IP:</label>
			<div class="col-xs-9"><input type="text" name="CS" id="CommanderServer" class="form-control"></div>
		</div>
		<div class="col-xs-offset-3 col-xs-9">
			<p><strong>Note:</strong> For the server IP, omit the <code>http://</code>. Example: <code>192.168.1.7:9876</code></p>
		</div>
		
		<div class="form-group">
			<label for="AuthKey" class="col-xs-3 control-label">Auth key:</label>
			<div class="col-xs-9"><input type="text" name="CP" id="AuthKey" class="form-control"></div>
		</div>
		<div class="form-group">
			<label for="outputMode" class="col-xs-3 control-label">Command output:</label>
			<div class="col-xs-9"><input type="checkbox" value="true" id="outputMode" class="form-control"></div>
		</div>
		<div class="form-group">
			<div class="col-xs-offset-3 col-xs-9">
				<input type="button" class="btn btn-primary" id="submit_button" value="Send to Pebble">
			</div>
		</div>
	</form>

	<div class="panel panel-warning">
		<div class="panel-body bg-warning">
			Re-launch the watch app for these settings to take affect.
		</div>
	</div>
</div>

<!-- send settings to pebble -->
<script>


function sendData() {
	var serverIP = document.getElementById("CommanderServer").value;
	var authKey = document.getElementById("AuthKey").value;
	var outputMode = document.getElementById("outputMode").checked;
	var options = {
		server: serverIP,
		authkey: authKey,
		commandoutput: outputMode
	}
	document.location = 'pebblejs://close#' + encodeURIComponent(JSON.stringify(options));
}

submitButton = document.getElementById('submit_button');
submitButton.addEventListener('click', function() {
	sendData();
});
</script>
</body>
</html>
```

Everything is a mess. I need to hire a maid. I want to have a working version up by the weekend. We will see how that goes. V2 will have the button configs but v1 will have scrollable commands.
