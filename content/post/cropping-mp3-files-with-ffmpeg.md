+++
title = "Cropping mp3 files with FFmpeg"
date = "2012-01-04"
+++
![](/img/ffmpeg-logo.png)

Today, I was trying to find a free app for cropping a mp3 sound file.
And I found a simple one with CLI. The [FFmpeg](http://ffmpeg.org/) is a
multimedia files handler and it is pretty complex. But to do this task
we will use the following parameters:

-   **-t** chop after specified number of seconds
-   **-ss** chop until specified number of seconds
-   **-acodec copy** to maintain encoding and sampling rate
-   **-i** use file as input file

And the final command was something like this:

` ffmpeg -acodec copy -y -t $start_at -ss $ends-at -i $inputfile.mp3 $outputfile.mp3`
