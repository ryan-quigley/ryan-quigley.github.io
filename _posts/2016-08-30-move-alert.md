---
layout: post
title: Hacking Together Activity Reminders
---
My current role as Data Science Intern requires me to be at a computer for the majority of my waking hours. I am definitely not complaining: I love programming and diving into data. That being said, I have tendency to get fully absorbed in the projects that I am working on. This often results in a time vortex and many sedentary hours, so it is important that I remember to get up and move on a regular basis. Since I am a dedicated Fitbit wearer, I looked to the Fitbit app for hourly activity reminders. The app allows users to set a range of 5-14 consecutive hours in a day for which Fitbit will track whether or not at least 250 steps were taken. Hourly activity reminders are available for the new Fitbit Alta, but not for the older models such as my device: the Charge HR; however, I would not be deterred. The plan:
1. Use the [fitbitScraper](https://github.com/corynissen/fitbitScraper) R package to get my activity data.
2. Figure out a way to notify myself if less than 250 steps were taken. Enter [Slackr](https://github.com/hrbrmstr/slackr).
3. Set up a crontab to run the R script while I am at work.

#### Complete R file
For the impatient, here is the complete R file:
```r
#!/Library/Frameworks/R.framework/Versions/3.3/Resources/bin/Rscript

## Libraries
library(slackr)
library(fitbitScraper)
library(lubridate)

## Slackr setup
slackrSetup(incoming_webhook_url = "https://hooks.slack.com/services/XXXXXXXXX/XXXXXXXXX/XXXXXXXXX")

## Fitbit code
cookie <- login(email = "your.email@gmail.com", password = Sys.getenv("FBPW"), rememberMe= TRUE)

d <- ymd_hms(Sys.time(), tz = "America/Los_Angeles")
h <- hour(d)
d.short <- as_date(d)
today <- get_intraday_data(cookie, what = "steps", date = as.character(d.short))
steps <- sum(today[hour(today$time) == h,2])

if (steps < 250) {
	slackrBot("One does not simply walk into Mordor...but you better start moving!", channel = "", username = "", incoming_webhook_url = Sys.getenv("SLACK_INCOMING_URL_PREFIX"))
}
```

#### [fitbitScraper](https://github.com/corynissen/fitbitScraper)
The two main functions I am using from this package are `login()` and `get_intraday_data()`. See the  [vignette](https://cran.r-project.org/web/packages/fitbitScraper/vignettes/fitbitScraper-examples.html) for more examples on how to use these functions. In the password argument of the `login()` function, I am passing in a system variable that I have set in my .Rprofile file. This file will be run anytime a new R session is started.

To do the steps analysis, I am also making use of the package [lubridate](https://github.com/hadley/lubridate) to make date and time handling easier.

Together these two packages allow me to determine the total number of steps recorded for the hour that the script is run.


#### [Slackr](https://github.com/hrbrmstr/slackr)
Since I always have slack open on my computer during working hours (as well as notifications enabled on my phone), it is the perfect channel for receiving notifications. I have no doubt it would have been a worthwhile endeavor to dig into the details of the slack API, but the slackr package was just too good to pass up.

Using this package requires the following:
1. Slack account
2. Set up incoming webhook url: [Slack API](https://api.slack.com/)

The two main functions being used from this package are `slackrSetup()` and `slackrBot()`. See the [documentation](https://cran.r-project.org/web/packages/slackr/slackr.pdf) for further details on these functions


#### Crontab
`env EDITOR=nano crontab -e`
```bash
50  09-18 * * 0-5 /path/to/R/file.R
```
`crontab -l`

#### Final R Stuff
Add this line to the top of R scripts so they can be run from the command line
```r
#!/Library/Frameworks/R.framework/Versions/3.3/Resources/bin/Rscript
```
Update permissions of R file: `chmod +x /path/to/R/file.R`


#### Final Result
![]({{ site.base_url}}/images/slack_move_alert.png)

#### Updating path to point to R version

The following needs to be added to the path variable via .bash_profile. In a terminal window enter the following command to edit .bash_profile: `nano ~/.bash_profile`. In the editor add the following lines to the bottom of .bash_profile:

```
export PATH='/Library/Frameworks/R.framework/Versions/3.3/Resources/bin:$PATH'
```
Hit ctrl+o to save, and ctrl+x to exit.

#### Next steps
I came up with this project several weeks into my summer data science internship. As a result, I have activity data for a number of weeks before and after implementing the activity reminders. I would like to apply some of the skills I learned in my design of experiments course during the spring. I want to determine whether or not the hourly activity reminder increased the number of hours in a day (between 9am and 7pm) where I took more than 250 steps. Since the day of the week is a known and controllable nuisance source of variability, I would like to use weekday as a blocking factor.
