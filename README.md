# slacktee #

*slacktee* is a bash script that works like [tee](http://en.wikipedia.org/wiki/Tee_(command)) command.
Instead of writing the standard input to files, *slacktee* posts it to [Slack](https://slack.com/).

![Image](https://github.com/course-hero/slacktee/blob/slacktee-readme-images/slacktee_demo.gif)

Requirements
------------

*slacktee* uses [curl](http://curl.haxx.se/) command to communicate with Slack.

Installation
------------

Please execute install.sh as shown below:
```
bash ./install.sh
```
install.sh copies slacktee.sh to `/usr/local/bin` and sets executable permission.
After the installation, interactive setup starts automatically.

Configuration
------------

Before start using *slacktee*, please set following variables in the script configuration file.
*slacktee* reads the global configuration (/etc/slacktee.conf) first, then reads your local configuration (~/.slacktee).
You can set up your local configuration file using interactive setup mode (--setup option).

For more details about tokens, visit [Slack's API page](https://api.slack.com/).

```
webhook_url=""      # Incoming Webhooks integration URL. See https://my.slack.com/services/new/incoming-webhook
upload_token=""     # The user's API authentication token, only used for file uploads. See https://api.slack.com/#auth
channel=""          # Default channel to post messages. '#' is prepended, if it doesn't start with '#' or '@'.
tmp_dir="/tmp"      # Temporary file is created in this directory.
username="slacktee" # Default username to post messages.
icon="ghost"        # Default emoji to post messages. You don't have to wrap it with ':'. See http://www.emoji-cheat-sheet.com.
attachment=""       # Default color of the attachments. If an empty string is specified, the attachments are not used.
```

Usage
------------
Feed input to *slacktee* through the pipe.

```
usage: slacktee.sh [options]
  options:
    -h, --help                        Show this help.
    -n, --no-buffering                Post input values without buffering.
    -f, --file                        Post input values as a file.
    -l, --link                        Add a URL link to the message.
    -c, --channel channel_name        Post input values to specified channel or user.
    -u, --username user_name          This username is used for posting.
    -i, --icon emoji_name             This icon is used for posting.
    -t, --title title_string          This title is added to posts.
    -m, --message-formatting format   Switch message formatting (none|link_names|full).
                                      See https://api.slack.com/docs/formatting for more details.
    -p, --plain-text                  Don't surround the post with triple backticks.
    -a, --attachment [color]          Use attachment (richly-formatted message)
                                      Color can be 'good','warning','danger' or any hex color code (eg. #439FE0)
                                      See https://api.slack.com/docs/attachments for more details.
    -e, --field title value           Add a field to the attachment. You can specify this multiple times
    -s, --short-field title value     Add a short field to the attachment. You can specify this multiple times
    --setup                           Set up .slacktee file interactively.
    --config filename.conf            Use the specified file for configuration.
```

If you'd like to post the output of `ls` command, you can do it like this.

```
ls | slacktee.sh
```

To post the output of `tail -f` command line by line, use `-n` option.

```
tail -f foobar.log | slacktee.sh -n
```

To post the output of `find` command as a file, use `-f` option.

```
find /var -name "foobar" | slacktee.sh -f
```

You can specify `channel`, `username`, `icon`, `title`, and `link` too.

```
ls | slacktee.sh -c "general" -u "slacktee" -i "shipit" -t "ls" -l "http://en.wikipedia.org/wiki/Ls"
```

Of course, you can connect another command with pipe.

```
ls | slacktee.sh | email "ls" foo@example.com
```

Would you like to use richly-formatted message? Use `-a`, `-e` and `-s` options.

```
cat error.log | slacktee.sh -a "danger" -e "Date and Time" "$(date)" -s "Host" "$(hostname)"
```

Direct message to your teammate 'chuck'? Easy!

```
echo "Submit Your Expense Reimbursement Form By Friday!" | slacktee.sh -c "@chuck"
```

You can find more examples on [Course Hero blog](http://www.coursehero.com/blog/2015/04/09/why-we-built-slacktee-a-custom-slack-integration/).

Travis-CI Integration
---------------------

You may want to integrate *slacktee* into Travis-CI in order to send additional
logging information to your Slack channel. In this case, it is recommended that
you **do not expose** your Incoming WebHook URL and API authentication token as
plaintext values inside your slacktee.conf file.

Instead, use the [encrypt command](https://github.com/travis-ci/travis.rb#encrypt)
of the Travis client to set the SLACKTEE\_WEBHOOK and SLACKTEE\_TOKEN
environment variables, and leave the *webhook_url* and *upload_token* values
in your slacktee.conf empty. When *slacktee* runs, it will give priority to the
environment variables, which Travis-CI will decrypt and set automatically during
the build process. In this way those two values are kept secure.

### Example

Modify slacktee.conf
```
webhook_url=""
upload_token=""
channel="Travis-CI"
tmp_dir="/tmp"
username="slacktee"
icon="ghost"
```

Add the encrypted environment variables to the .travis.yml file in your git
repository
```bash
travis encrypt SLACKTEE_WEBHOOK='https://hooks.slack.com/services/afternoonTEE/BMP2vsT72/ohNoDontTellUs' --add
travis encrypt SLACKTEE_TOKEN='yoho-0987654321-1234567890-4488116622-abc123' --add
```

Looking at your travis.yml you will now see the following added
```yaml
env:
  global:
  - secure: 2YZabH8+UdzqyBWckojRDP9uudnCSYyxOOx1y85el69YdHwLDMD+dt49rAgIrmCWsWCWpUZ0ZRWV8vU2VFMffIhmikiqG7VoKHuN5PyY8qBwr9hq/ZI8gdwgjgfRIGtv/U89BTjMmc1g/6nJkSvMtiSUSK3Lopg0JCyuZsiyhzs=
  - secure: TKpohmywdOneQkqGxJiF+S1N8oCdTWWGsXgjzNXWSvb23KDtvGq/W2SpWdFdwEHC9Y8NymoAPYRSW8MUQoiJ7NaQ1eZQuyx6/orjHpIgqiAuHrOSaMagzpKVG6Gtb87qDgov65ZOasyex1OtPQdfFtZBX67B6IVXkRPV+IA/+UX=
```

An example travis.yml section using *slacktee* may look like:
```yaml
after_failure:
- ls /path/to/build | ./slacktee.sh -t "$TRAVIS_REPO_SLUG $TRAVIS_JOB_NUMBER build directory"
- cat /path/to/some.log | ./slacktee.sh -t "$TRAVIS_REPO_SLUG $TRAVIS_JOB_NUMBER some.log"
```
