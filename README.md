# "render_time_lapse.py"

## Description: 
Python application used to process photographic
	frames and render them into a video clip based on
	user preferences. Designed for use with time lapse
	photography and can also be used for stop-motion
	animation. Features include deflickering,
	"Ken Burns"-style crop/pan/zoom effects, dynamic 
	brightening, and 4K HD support.

## Dependencies:
	PIL (pillow), os, sys, glob, time, numpy, json, tqdm

## How to:

### Basic Project Setup:
	First, create a project directory with the following structure:
```
/{project_name}/
|	proj_config.json
|
|______	orig_frames/
|	|	{frames}.{filetype}
```
	Where the photographic frames you wish to render to a video
	are contained within the /orig_frames/ folder in your project
	directory. Note that the names of the files must be ordered alpha-numerically
	according to their timing within the sequence. If these files come
	directly from a camera, this requirement should already be satisfied.
	The "proj_config.json" file can be copied from examples
	contained within this git repository under /proj_config_examples/.
	Be sure to rename the example to "proj_config.json." This file contains information about
	how you want the video to be rendered. An example proj_config.json 
	file is shown below:
```
{ 
	"filetype" : "JPG",
	"size" : [3840, 2160],
	"frame_rate" : 30,
	"bit_rate" : "45000k",
	"deflicker" : false,
	"df_window" : 15,
	"bright" : null,
	"crop" : null
}
```
	The "filetype" field sets the expected file extension for the photo frames.
	The fields "size," "frame_rate," and "bit_rate" set parameters for the final 
	rendered video. Recommended values are given in the /proj_config_examples/
	subdirectory for 30 fps 1080p HD and 4K UHD. 

### Deflickering:
	"Deflicker" indicates whether or not you want to apply deflickering processing to your frames, useful
	if there are random brightness variations between individual frames. Type 'true'
	to apply deflickering, 'false' for no application of deflickering. Note that
	the first time you perform a deflickering analysis it may add significant
	processing time. Information is then stored in a file named "brightness_orig_frames.csv,"
	contained within the project directory, which speeds processing on subsequent runs.
	"df_window" indicates the width of a window, in frames, used to calculate
	mean brightness of frames over time to apply deflickering.

### Brightness Keyframes:
	In the above example proj_config.json file, "bright" is set to 'null,' indicating
	no desired change in brightness. One can alter the overall brightness of all frames
	by setting "bright" to a value which multiplicatively alters the brightness of the scene,
	i.e. 1.0 indicates no change, 2.0 is twice as bright, 0.5 is half as bright.
	To slightly brighten the entire scene, use a value such as:
```
	"bright" : 1.1
```
	You can also enter an array of brightness "keyframes" that change the value 
	dynamically throughout the sequence. A keyframe index indicates the photographic
	frame to which to apply the specific brightness value, while the program interpolates
	between values. The format for keyframes is as follows:
```
	"bright" : [ 
		[index1, brightness1], 
		[index2, brightness2] 
		]
``` 
	Not that negative indices can be used, with -1 indicating the final frame. As an
	example, if you had a time lapse of a sunset and wanted to darken the beginning of 
	the scene and brighten the ending of the scene, with a smooth transition between them,
	you might try the following:
```
	"bright" : [
		[0, 0.8],
		[-1, 1.2]
		]
```

### Cropping Keyframes:
	The "crop" field works much the same way as the "bright" field. If "crop" is set to
	'null,' the program automatically crops the photos (letterbox style) to match the 
	output video size. A single crop value can be set in the following format:
```
	"crop" : [x, y, X, Y]
```
	where (x,y) is are the coordinates of the upper left corner and (X,Y) those of the 
	lower right corner of the crop. NOTE: the practitioner is responsible for getting the aspect ratio correct, otherwise
	distortion may occur. Crop keyframe values can also be set by introducing the
	keyframe indices at the beginning of the row for each set of crop values:
```
	"crop" : [ 
		[index1, x1, y1, X1, Y1],
		[index2, x2, y2, X2, Y2],
		]
```
	As an example, the following set of crop keyframes produces a "Ken Burns" effect of zooming in
	to the center-right portion of the frame between frames 60 & 300:
```
	"crop" : [
		[60, 0, 158, 3006, 1848],
		[300, 1150, 456, 2970, 1480]
		]
```
### Rendering the video:
	Once you have your "proj_config.json" file configured the way you want and your
	frames in the appropriate directory, you can render your video using the following command:
```
python render_time_lapse.py {project_directory}
```
	Edited photo frames will be placed in {project_directory}/rendered_frames/ and the
	final video will be saved to {project_directory}/rendered_video.mp4 in the
	mpeg4 format.
