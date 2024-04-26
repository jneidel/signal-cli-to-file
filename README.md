# signal-cli-to-file

> Save incoming signal messages as files (for your note-taking system)

This script will parse all incoming messages (and attachments) an create files
out of them in a specified location.

My use case is to write notes on my phone, take pictures or record audios that
then just show up in my note-taking system. All very convieniently through
signal.

## signal-cli providers, or: What is the difference between the two scripts?

There are two possible ways to interact with signal-cli.

- [signal-cli](https://github.com/AsamK/signal-cli) - full functionality, invoke when needed, structured text output
- [signal-cli-rest-api](https://github.com/bbernhard/signal-cli-rest-api) - always running, lacks some features, JSON interface

The repo provides script for parsing either of them.
(I switched to the rest-api, since my local signal-cli broke.)

- [signal-cli version](signal-cli-to-inbox)
- [signal-cli-rest-api version](signal-api-to-inbox)

## About the scripts

They were created for my specific use, so you might not like some of the
opinions.

### Principles & Quirks

- One message = one file
- Everything is written to one single "inbox" directory
- Errors are also written into the output directory (you quickly see if something goes wrong)
- In case of naming collision: append instead of overwrite
- Filenames past 60 characters are shortend
- A colon (`:`) in the first line of the message to specify the file name
- If there is a message together with an attachment that will be the title
- Single script file, no dependencies

While the scripts generally work, bugs or some misbehaviors are to be expected.

### Intended usage

The script is intended to be run via cron (errors written to files as well).
Nothing stands in the way of triggering manually, you just won't get any output.

## Setup

The script themselves require the phone number you want to use to be setup in
the chosen provider.

Then configure some things about your local system.

### Setting up your number in signal-cli

Follow the instruction in the respective project:
- [signal-cli](https://github.com/AsamK/signal-cli?tab=readme-ov-file#usage)
- [signal-cli-rest-api](https://github.com/bbernhard/signal-cli-rest-api?tab=readme-ov-file#getting-started)

My notes for registering with a landline number:

```sh
# generate captcha: https://signalcaptchas.org/registration/generate
############### signal-cli setup
signal-cli -a $SIGNAL_NUMBER register --captcha CAPTCHA
sleep 60s
signal-cli -a $SIGNAL_NUMBER register --voice --captcha CAPTCHA
signal-cli -a $SIGNAL_NUMBER verify CODE
signal-cli -a $SIGNAL_NUMBER updateProfile  --given-name "My" --family-name "Bot" --about "Beep Boop, I'm automated" --avatar inbox.png

############### signal-api setup
# api ref: https://bbernhard.github.io/signal-cli-rest-api
curlj POST $API_HOST/v1/register/$SIGNAL_NUMBER '{use_voice: false, captcha: "CAPTCHA"}'
sleep 60s
curlj POST $API_HOST/v1/register/$SIGNAL_NUMBER '{use_voice: true, captcha: "CAPTCHA"}'
curlj POST $API_HOST/v1/register/$SIGNAL_NUMBER/verify/TOKEN
curlj PUT  $API_HOST/v1/profiles/$SIGNAL_NUMBER "{ name: 'My Bot', base64_avatar: '$(cat inbox.png | base64 -w0 -)' }"
curlj POST $API_HOST/v2/send "{number: '$SIGNAL_NUMBER', message: 'Hi from the API', recipients: ['YOUR_NUMBER']}'
```

- [Captcha explaination](https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha)
- [curlj script](https://github.com/jneidel/dotfiles/blob/master/scripts/curlj)

### Configuring the scripts

The scripts themselves contain option description and examples.

## Tested scenarios
### signal-api-to-inbox

I jotted down these cases while building and testing the script:

- Text
- Text: multi-line
- Text: only url
- Text: with colon to be used as title
- Attachment: image
- Attachment: multiple images
- Attachment: audio recording
- Attachment: pdf where the original file name is used
- Attachment: with a message to be used as the file name
