# qlcplus-msc
Adding support for controlling QLC+ with MIDI Show Control

#### What's this?

This is a kind-of plugin (ehm.. hack) to make the software [QLC+](https://www.qlcplus.org/), a popular app to control lighting, respond to [MIDI Show Control](https://en.wikipedia.org/wiki/MIDI_Show_Control) command. (MSC being an old and reliable protocol for show automation)

#### Why?

Show automation is a way to make the technical execution of a show (be it a theatrical performance, concert or some other event) more reliable. How? Well, by removing most of the human element.

MSC is supported by a vast majority of lighting consoles but this is sadly missing from QLC+

#### How does this work?

This script is a snippet of HTML (mainly some javascript) which the user pastes into a "Virtual console label". The virtual console is subsequently accessed from a browser that supports the [Web MIDI API](https://caniuse.com/midi) and the processing of these commands also happens here.

## Usage

Download or open the html file included in this repository and copy the entire file contents to your clipboard. In your existing QLC+ project create a new virtual console label and paste the copied code. Resize the label so you'd be able to read about two lines of text when in operation.

Open the QLC+ web interface from the device which will recieve the MSC commands. (Yes, it can be a different device as long as it can open the web interface). For more information on what is needed to enable this feature, please refer to the documentation [here](https://www.qlcplus.org/docs/html_en_EN/webinterface.html)

When prompted give the browser tab permissions to use MIDI devices. Without this it won't work.

#### Customzing options

The script starts out with some varables which change some of the scripts behavior:

`msc_device_id` - set the MSC device ID. This can be a number between 0 and 126. This script will always respond to device id 127 (all-call) as per the MSC specifications.

`autoload` - if set to `true` the scipt will run when the page is loaded. Otherwise the script is run when the user doubleclicks the label

`convert_cue_numbers` - if set to `true` the script will derive new cue numbers from the name of the cue. See below why this might be usefull.

## Sending MSC commands to QLC+

This script responds to two MSC commands:

#### `GO`

- If no cue number is specified, this script will advance *all* cue lists to the next cue (quite like pressing the "next cue" button).
- If only a cue number is specified, this script will start the fade in of all cues which match by number (in all cue lists).
- If a cue number and list is specified, only the cues matching in number and list are fired

If at any point multiple about-to-be-fired cues share the same (generated) number, only the first one in each cue list actually gets fired.

***Note:** The number of the cue list is the ID of the virtual console element. For convenience this is displayed in the top-left cell of each cue list. This can't really be changed unless you modify the .qxw file with a text editor*

#### `RESET`

Stops *all* cue lists. MSC does not provide facilities to specify wich cue list to stop.

The MSC specifications do specify other types of commands, however, they either aren't relevent to the functions of QLC+ or they can't be implemented per spec due to limitations in QLC+ (e. g. STOP and RESUME can't be implemented, because they share a single API call whose result is unfortunately state dependent.)

---

#### A word on cue numbers

QLC+ quite unlike most other equipment uses only positive-intigers to number cues. The industry standard is to allow the use of numbers with a decimal point (e. g. `15.1` or `27.35`). The reason is that if you create a cue that belongs in between two existing ones (perhaps you forgot about it initially or the need for it came later) all cues after the new one would have their numbers changed. This is a major complication if you have the old cue numbers noted on paper or if other devices are set to fire preprogammed cue numbers that would suddenly not be correct.

MSC transmits cue numbers as characters. Permissible characters are numbers (0-9) and a period symbol. This script can convert cue names to numbers by removing all non-supported characters and converting dashes and underscores to periods. This is because the cue \[scene\] name is the pretty much the only user-definable value that can be used for this purpouse. It is still, however, possible to give cues custom names to, for instance describe what they do. For example: `15.1 blackout` or `27.35 fade cyc to 25%`.

#### Why does this need to be done externally?

It doesn't. I hope someday my code becomes obsolete. I created this tool because I needed it to exist and don't have the expertise to write C++. It's kinda a hack to be honest. This implementation is vulnerable to the browser caching the site and pausing JS execution if it thinks the tab is inactive. Native support would of course be preferable. Improvements to the way this works currently could possibly be done by runing this code outside the browser in Node.js and better utilizing the QLC+ Web API.
