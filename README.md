# breach-data-storage
*Adventures in processing and storing massive amounts of data*

## What this is
Investigators use any suitable public data for their OSINT investigations. For some of us, this includes breach data, i.e. information released publicly to forums, communities, and channels. A few years ago, it was still possible to obtain a few of these data sets, place them on a fast storage device, and search through them with `ripgrep.` This is no longer the case with daily leaks and major sets reaching upwards of TB of data each.

This repo contains my efforts to organize and optimize this type of data for fast and flexible querying. 


## What this isn't
This repo does not and will not at any point contain actual data from any breach. It is not intended to encourage any illicit activity. Also, it is not the final word on database design, data transformation, or storage. We are all learning as we go. If you have a better way of doing any of this – reach out and contribute!

## Why
With services like HIBP, Dehashed, IntelX, and others, you may wonder why would you ever bother doing this yourself. The answer is it depends. It will take a lot of time to sift through data and organize it. It will likely need money and willingness to spend it on fast data devices. And it will take a mindset to learn and persist through challenges. In return you get a few key benefits: privacy, greater depth of data, and full control of the data.

# Goals
- fast querying of data on key fields, such as `name`, `username`, `email`, `password`, `phone`, `ip`
- querying of additional data that may enrich the above
- source for the above data
- low maintenance overhead
- ease of backup/restore

# Tools
[Tools](tools.md) used to process the data

# Process

