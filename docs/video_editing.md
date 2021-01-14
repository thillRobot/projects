#video_editing

This readme is a general guide to video editing and processing. 
 
This needs work! 

## using ffmpeg to compress videos for upload

Run this in the terminal where the files 'raw' video files are.

$ ffmpeg -i input.mp4 -vcodec libx264 -crf 28 output.mp4
