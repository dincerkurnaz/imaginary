# imaginary [![Build Status](https://travis-ci.org/h2non/imaginary.png)](https://travis-ci.org/h2non/imaginary) [![GitHub release](https://img.shields.io/github/tag/h2non/imaginary.svg)](https://github.com/h2non/imaginary/releases) [![GoDoc](https://godoc.org/github.com/h2non/imaginary?status.png)](https://godoc.org/github.com/h2non/imaginary) [![Coverage Status](https://coveralls.io/repos/h2non/imaginary/badge.svg?branch=master)](https://coveralls.io/r/h2non/imaginary?branch=master)

<img src="https://github.com/h2non/imaginary/blob/master/fixtures/imaginary.jpg" width="240" align="right" />

Simple and fast HTTP microservice for image processing powered by [bimg](https://github.com/h2non/bimg) and [libvips](https://github.com/jcupitt/libvips). Think about imaginary as private or public HTTP service for massive image processing/resizing.

It supports a common set of [image operations](#supported-image-operations) exposed as a simple [HTTP API](#http-api), 
with additional support for API token-based authentication, gzip compression and CORS support for direct web browser access.

It can read JPEG, PNG, WEBP and TIFF formats and output to JPEG, PNG and WEBP, including conversion between them. 
It supports common [image operations](#supported-image-operations) such as crop, resize, rotate, zoom, watermark... 
For getting started, take a look to the [HTTP API](#http-api) documentation.

imaginary uses internally libvips, a powerful library written in C for binary image processing which requires a [low memory footprint](http://www.vips.ecs.soton.ac.uk/index.php?title=Speed_and_Memory_Use) and it's typically 4x faster than using the quickest ImageMagick and GraphicsMagick settings or Go  native `image` package, and in some cases it's even 8x faster processing JPEG images. 

**Note**: imaginary is still beta. Do not use in compromised environments yet

## Prerequisites

- [libvips](https://github.com/jcupitt/libvips) v7.40.0+ (7.42.0+ recommended)
- C compatible compiler such as gcc 4.6+ or clang 3.0+
- Go 1.3+

## Installation

```bash
go get -u gopkg.in/h2non/imaginary.v0
```

### libvips

Run the following script as `sudo` (supports OSX, Debian/Ubuntu, Redhat, Fedora, Amazon Linux):
```bash
curl -s https://raw.githubusercontent.com/lovell/sharp/master/preinstall.sh | sudo bash -
```

The [install script](https://github.com/lovell/sharp/blob/master/preinstall.sh) requires `curl` and `pkg-config`

### Docker

```
docker pull h2non/imaginary
```

See [Dockerfile](https://github.com/h2non/imaginary/blob/master/Dockerfile) for more details.

### Heroku 

Required buildpacks. Add them in `.buildpacks`:
```
https://github.com/alex88/heroku-buildpack-vips.git
https://github.com/kr/heroku-buildpack-go.git
```

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
- Info (image size, format, orientation, alpha...)

## Performance

libvips is probably the faster open source solution for image processing. 
Here you can see some performance test comparisons for multiple scenarios:

- [libvips speed and memory usage](http://www.vips.ecs.soton.ac.uk/index.php?title=Speed_and_Memory_Use)
- [sharp performance tests](https://github.com/lovell/sharp#the-task) 
- [bimg](https://github.com/h2non/bimg#Performance) (Go library with C bindings to libvips)

## Usage

```
imaginary server

Usage:
  imaginary -p 80
  imaginary -cors -gzip
  imaginary -h | -help
  imaginary -v | -version

Options:
  -a <addr>     bind address [default: *]
  -p <port>     bind port [default: 8088]
  -h, -help     output help
  -v, -version  output version
  -cors         Enable CORS support [default: false]
  -gzip         Enable gzip compression [default: false]
  -key <key>    Define API key
  -cpus <num>   Number of used cpu cores.
                (default for current machine is 8 cores)
```

Start the server on a custom port
```bash
imaginary -p 8080
```

Also, you can pass the port as environment variable
```bash
POST=8080 imaginary 
```

Enable debug mode
```
DEBUG=* imaginary -p 8080
```

Or filter debug output by package
```
DEBUG=imaginary imaginary -p 8080
```

## HTTP API

### Authorization

imaginary supports a simple token-based API authorization. 
To enable it, you should specific the flag `-key secret` when you call the binary.

API token can be defined as HTTP header (`API-Key`) or query param (`key`).

Example request with API key:
```
POST /crop HTTP/1.1
Host: localhost:8088
API-Key: secret
```

#### GET /form
Content Type: `text/html`

Serves an ugly HTML form, just for testing/playground purposes

#### POST /info
Accept: `image/*, multipart/form-data`. Content-Type: `application/json` 

Returns the image metadata as JSON

#### POST /crop
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

Crop the image by a given width or height. Image ratio is maintained

##### Allowed params

- width `int`
- height `int`

##### Required params

- `width` or `height`

#### POST /resize
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

Resize an image by width or height. Image aspect ratio is maintained

##### Allowed params

- width `int`
- height `int`

##### Required params

- `width` or `height`

#### POST /enlarge
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /extract
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /zoom
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /thumbnail
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /rotate
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /flip
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /flop
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /convert
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

#### POST /watermark
Accept: `image/*, multipart/form-data`. Content-Type: `image/*` 

## License

MIT - Tomas Aparicio
