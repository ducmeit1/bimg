# bimg [![Build Status](https://travis-ci.org/h2non/bimg.png)](https://travis-ci.org/h2non/bimg) [![GitHub release](https://img.shields.io/github/tag/h2non/bimg.svg)](https://github.com/h2non/bimg/releases) [![GoDoc](https://godoc.org/github.com/h2non/bimg?status.png)](https://godoc.org/github.com/h2non/bimg) [![Coverage Status](https://coveralls.io/repos/h2non/bimg/badge.svg?branch=master)](https://coveralls.io/r/h2non/bimg?branch=master)

Small [Go](http://golang.org) library for blazing fast and efficient image processing based on [libvips](https://github.com/jcupitt/libvips) using C bindings. It provides a clean, simple and fluent [API](https://godoc.org/github.com/h2non/bimg) in pure Go.

bimg is designed to be a small and efficient library with a generic and useful set of features. 
It uses internally libvips, which requires a [low memory footprint](http://www.vips.ecs.soton.ac.uk/index.php?title=Speed_and_Memory_Use) 
and it's typically 4x faster than using the quickest ImageMagick and GraphicsMagick settings or Go  native `image` package, and in some cases it's even 8x faster processing JPEG images. 

It can read JPEG, PNG, WEBP and TIFF formats and output to JPEG, PNG and WEBP. It supports common [image transformation](#supported-image-operations) operations such as crop, resize, rotate, zoom, watermark... and conversion between multiple formats. 

For getting started, take a look to the [examples](#examples) and [programmatic API](https://godoc.org/github.com/h2non/bimg) documentation.

bimg was heavily inspired in [sharp](https://github.com/lovell/sharp), 
its homologous package built for node.js by [Lovell Fuller](https://github.com/lovell).

**Note**: bimg is still beta. Pull request and issues are highly appreciated

## Prerequisites

- [libvips](https://github.com/jcupitt/libvips) v7.40.0+ (7.42.0+ recommended)
- C compatible compiler such as gcc 4.6+ or clang 3.0+
- Go 1.3+

## Installation

```bash
go get -u gopkg.in/h2non/bimg.v0
```

### libvips

Run the following script as `sudo` (supports OSX, Debian/Ubuntu, Redhat, Fedora, Amazon Linux):
```bash
curl -s https://raw.githubusercontent.com/lovell/sharp/master/preinstall.sh | sudo bash -
```

The [install script](https://github.com/lovell/sharp/blob/master/preinstall.sh) requires `curl` and `pkg-config`

## Supported image operations

- Resize
- Enlarge
- Crop
- Rotate (with auto-rotate based on EXIF orientation)
- Flip (with auto-flip based on EXIF metadata)
- Flop
- Zoom
- Thumbnail
- Extract area
- Watermark (fully customizable text-based)
- Format conversion (with additional quality/compression settings)
- EXIF metadata (size, alpha channel, profile, orientation...)

## Performance

libvips is probably the faster open source solution for image processing. 
Here you can see some performance test comparisons for multiple scenarios:

- [libvips speed and memory usage](http://www.vips.ecs.soton.ac.uk/index.php?title=Speed_and_Memory_Use)
- [sharp performance tests](https://github.com/lovell/sharp#the-task) 

#### Benchmarks

Tested using Go 1.4 and libvips-7.42.3 in OSX i7 2.7Ghz
```
PASS
BenchmarkResizeLargeJpeg  30    46652408 ns/op
BenchmarkResizePng        20    57387902 ns/op
BenchmarkResizeWebP       500    2453220 ns/op
BenchmarkConvertToJpeg    30    35556414 ns/op
BenchmarkCrop             30    51768475 ns/op
BenchmarkExtract          30    50866406 ns/op
ok 9.424s
```

## API

### Examples

```go
import (
  "fmt"
  "os"
  "gopkg.in/h2non/bimg.v0"
)
```

#### Resize

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Resize(800, 600)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

size, err := bimg.NewImage(newImage).Size()
if size.Width == 400 && size.Height == 300 {
  fmt.Println("The image size is valid")
}

bimg.Write("new.jpg", newImage)
```

#### Rotate

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Rotate(90)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Convert

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Convert(bimg.PNG)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

if bimg.NewImage(newImage).Type() == "png" {
  fmt.Fprintln(os.Stderr, "The image was converted into png")
}
```

#### Custom options

See [Options](https://godoc.org/github.com/h2non/bimg#Options) struct to discover all the available fields

```go
options := bimg.Options{
  Width:        800,
  Height:       600,
  Crop:         true,
  Quality:      95,
  Rotate:       180,
}

buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Process(options)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Watermark

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

options := bimg.Watermark{
  Watermark{
    Text:       "Chuck Norris (c) 2315",
    Opacity:    0.25,
    Width:      200,
    DPI:        100,
    Margin:     150,
    Font:       "sans bold 12",
    Background: bimg.Color{255, 255, 255},
  }
}

newImage, err := bimg.NewImage(buffer).Watermark()
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Fluent interface

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

image := bimg.NewImage(buffer)

// first crop image
_, err := image.CropByWidth(300)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

// then flip it
newImage, err := image.Flip()
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

// save the cropped and flipped image
bimg.Write("new.jpg", newImage)
```

#### func  DetermineImageTypeName

```go
func DetermineImageTypeName(buf []byte) string
```
Determines the image type format by name (jpeg, png, webp or tiff)

#### func  Initialize

```go
func Initialize()
```
Explicit thread-safe start of libvips. You should only call this function if you
previously shutdown libvips

#### func  IsTypeNameSupported

```go
func IsTypeNameSupported(t string) bool
```
Check if a given image type name is supported

#### func  IsTypeSupported

```go
func IsTypeSupported(t ImageType) bool
```
Check if a given image type is supported

#### func  Read

```go
func Read(path string) ([]byte, error)
```

#### func  Resize

```go
func Resize(buf []byte, o Options) ([]byte, error)
```

#### func  Shutdown

```go
func Shutdown()
```
Explicit thread-safe libvips shutdown. Call this to drop caches. If libvips was
already initialized, the function is no-op

#### func  VipsDebug

```go
func VipsDebug()
```
Output to stdout collected data for debugging purposes

#### func  Write

```go
func Write(path string, buf []byte) error
```

#### type Angle

```go
type Angle int
```


```go
const (
  D0   Angle = C.VIPS_ANGLE_D0
  D90  Angle = C.VIPS_ANGLE_D90
  D180 Angle = C.VIPS_ANGLE_D180
  D270 Angle = C.VIPS_ANGLE_D270
)
```

#### type Direction

```go
type Direction int
```


```go
const (
  HORIZONTAL Direction = C.VIPS_DIRECTION_HORIZONTAL
  VERTICAL   Direction = C.VIPS_DIRECTION_VERTICAL
)
```

#### type Gravity

```go
type Gravity int
```


```go
const (
  CENTRE Gravity = iota
  NORTH
  EAST
  SOUTH
  WEST
)
```

#### type Image

```go
type Image struct {
}
```


#### func  NewImage

```go
func NewImage(buf []byte) *Image
```
Creates a new image

#### func (*Image) Convert

```go
func (i *Image) Convert(t ImageType) ([]byte, error)
```
Convert image to another format

#### func (*Image) Crop

```go
func (i *Image) Crop(width, height int, gravity Gravity) ([]byte, error)
```
Crop the image to the exact size specified

#### func (*Image) CropByHeight

```go
func (i *Image) CropByHeight(height int) ([]byte, error)
```
Crop an image by height (auto width)

#### func (*Image) CropByWidth

```go
func (i *Image) CropByWidth(width int) ([]byte, error)
```
Crop an image by width (auto height)

#### func (*Image) Enlarge

```go
func (i *Image) Enlarge(width, height int) ([]byte, error)
```
Enlarge the image from the by X/Y axis

#### func (*Image) Extract

```go
func (i *Image) Extract(top, left, width, height int) ([]byte, error)
```
Extract area from the by X/Y axis

#### func (*Image) Flip

```go
func (i *Image) Flip() ([]byte, error)
```
Flip the image about the vertical Y axis

#### func (*Image) Flop

```go
func (i *Image) Flop() ([]byte, error)
```
Flop the image about the horizontal X axis

#### func (*Image) Metadata

```go
func (i *Image) Metadata() (ImageMetadata, error)
```
Get image metadata (size, alpha channel, profile, EXIF rotation)

#### func (*Image) Process

```go
func (i *Image) Process(o Options) ([]byte, error)
```
Transform the image by custom options

#### func (*Image) Resize

```go
func (i *Image) Resize(width, height int) ([]byte, error)
```
Resize the image to fixed width and height

#### func (*Image) Rotate

```go
func (i *Image) Rotate(a Angle) ([]byte, error)
```
Rotate the image by given angle degrees (0, 90, 180 or 270)

#### func (*Image) Size

```go
func (i *Image) Size() (ImageSize, error)
```
Get image size

#### func (*Image) Thumbnail

```go
func (i *Image) Thumbnail(pixels int) ([]byte, error)
```
Thumbnail the image by the a given width by aspect ratio 4:4

#### func (*Image) Type

```go
func (i *Image) Type() string
```
Get image type format (jpeg, png, webp, tiff)

#### type ImageMetadata

```go
type ImageMetadata struct {
  Orientation int
  Channels    int
  Alpha       bool
  Profile     bool
  Type        string
  Space       string
  Size        ImageSize
}
```


#### func  Metadata

```go
func Metadata(buf []byte) (ImageMetadata, error)
```
Extract the image metadata (size, type, alpha channel, profile, EXIF
orientation...)

#### type ImageSize

```go
type ImageSize struct {
  Width  int
  Height int
}
```


#### func  Size

```go
func Size(buf []byte) (ImageSize, error)
```
Get the image size by width and height pixels

#### type ImageType

```go
type ImageType int
```


```go
const (
  UNKNOWN ImageType = iota
  JPEG
  WEBP
  PNG
  TIFF
  MAGICK
)
```

#### func  DetermineImageType

```go
func DetermineImageType(buf []byte) ImageType
```
Determines the image type format (jpeg, png, webp or tiff)

#### type Interpolator

```go
type Interpolator int
```

```go
const (
  BICUBIC Interpolator = iota
  BILINEAR
  NOHALO
)
```

#### func (Interpolator) String

```go
func (i Interpolator) String() string
```

#### type Options

```go
type Options struct {
  Height       int
  Width        int
  AreaHeight   int
  AreaWidth    int
  Top          int
  Left         int
  Extend       int
  Quality      int
  Compression  int
  Crop         bool
  Enlarge      bool
  Embed        bool
  Flip         bool
  Flop         bool
  Rotate       Angle
  Gravity      Gravity
  Type         ImageType
  Interpolator Interpolator
}
```

## Special Thanks

- [John Cupitt](https://github.com/jcupitt)

## License

MIT - Tomas Aparicio
