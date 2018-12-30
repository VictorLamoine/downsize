# thumbsup-downsize

> Convert / resize / transcode / down-sample photos & videos to be web-friendly

This is one of the core modules of [thumbsup.github.io](https://thumbsup.github.io).

[![NPM](http://img.shields.io/npm/v/thumbsup-downsize.svg?style=flat-square)](https://npmjs.org/package/thumbsup-downsize)
[![License](http://img.shields.io/npm/l/thumbsup-downsize.svg?style=flat-square)](https://github.com/thumbsup/thumbsup-downsize)
[![Build Status](http://img.shields.io/travis/thumbsup/downsize.svg?style=flat-square)](http://travis-ci.org/thumbsup/downsize)
[![Dependencies](http://img.shields.io/david/thumbsup/downsize.svg?style=flat-square)](https://david-dm.org/thumbsup/downsize)
[![Dev dependencies](http://img.shields.io/david/dev/thumbsup/downsize.svg?style=flat-square)](https://david-dm.org/thumbsup/downsize)
[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](http://standardjs.com/)

## Setup

```bash
npm install thumbsup-downsize --save
```

This module requires the following binaries available in the system path:

- [GraphicsMagick](http://www.graphicsmagick.org/) for processing images
- [FFMpeg](https://ffmpeg.org/) for processing videos
- [Gifsicle](http://www.lcdf.org/gifsicle/) for processing animated GIFs

To run the tests, you will also need

- [ExifTool](http://www.sno.phy.queensu.ca/~phil/exiftool/)

## Usage

```js
const downsize = require('thumbsup-downsize')

const options = { height: 100, width: 100 }
downsize.image('source.tiff', 'thumb.jpg', options, (err) => {
  console.log(err || 'Thumbnail created successfully')
})
```

## API

### .image

```js
.image(source, target, options, callback)
```

Processes the image in `source` and creates a new image in `target`.
The image is appropriately converted if needed based on the `target` file extension.
You can specify the following options:

##### Image size

```js
// proportionally resize the photo to a maximum height
opts = { height: 300 }

// proportionally resize the photo to a maximum width
opts = { width: 300 }

// resize and crop the photo to exactly height x width
// the image will not be distorted
opts = { height: 100, width: 100 }
```

##### Image quality

```js
// quality between 0 and 100
opts = { quality: 80 }
```

##### Watermark

You can overlay a transparent watermark over the final image:

```js
opts = {
  watermark: {
    file: 'path/watermark.png',  // transparent PNG
    position: 'NorthEast'        // position of the watermark
  }
}
```

The possible values for `position` are:

- `Repeat` to repeat the watermark across the whole image
- `Center` to position the watermark in the middle
- `NorthWest`, `North`, `NorthEast`, `West`, `East`, `SouthWest`, `South`, `SouthEast` to position the watermark along the edge

![watermark](docs/watermark.jpg) ![tiled](docs/watermark-tiled.jpg)

Note: watermarks are not compatible with cropped images.
The `watermark` option will simply be ignored if both width and height are specified.

##### Post-processing

You can specify extra arguments that will be passed to GraphicsMagick.
This only works with [output arguments](https://github.com/aheckmann/gm#custom-arguments).

```js
opts = {
  args: [
    '-unsharp 2 0.5 0.7 0',
    '-modulate 120'
  ]
}
```

##### GIF animation

By default, only the first frame of an animated GIF is exported.
You can keep the entire animation by specifying:

```js
opts = { animated: true }
```

This offloads the processing of the image to [Gifsicle](https://github.com/kohler/gifsicle).
Note that:

- The destination file extension *must* be `.gif`
- The only other supported parameters are `width` and `height` (e.g. no watermarks)
- Cropping (specifying *both* width and height) is not supported and will throw an error

The flag is simply ignored if the source file is not a GIF.

### .still

```js
.still(source, target, options, callback)
```

Extract a single frame from the video in `source`, and writes the image to `target`.
This method supports all  the same options as `.image()`.

### .video

```js
.video(source, target, options, callback)
```

Transcodes the video in `source` to a web-friendly format and lower bitrate, and writes it in `target`.

#### format

The default export format is mp4. You can specify an export format by adding a `format` option whose value can be `mp4` or `webm`.

```js
opts = { format: webm }
```

#### bitrate

The default export bitrate depends on the codec used. You can specify an export bitrate by adding a `bitrate` option. Check [ffmpeg docmentation](https://trac.ffmpeg.org/wiki/Encode/H.264) for more information.

```js
opts = { bitrate: xxx }
```

The `.video()` call returns an [EventEmitter](https://nodejs.org/api/events.html)
to follow the progress of the conversion, since it can take a long time.

```js
const emitter = downsize.video(/* ... */)
emitter.on('progress', percent => console.log(`${percent}%`))
```

## Contributing

Image/video resizing is hard to unit test.
Instead, this repo contains an integration test suite made of many different resized files,
covering different file formats and edge cases.

When submitting a change, make sure you run the build locally.

```bash
npm test
```

If you don't have all dependencies installed, you can also run the tests in Docker.

```bash
docker build -t downsize-test .
docker run downsize-test
```
