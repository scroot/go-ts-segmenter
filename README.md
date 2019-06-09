# go-ts-segmenter
This tool enables you to segment (create HLS playable chunks) a live transport stream that is read from `stdin`. It also creates in real time the the HLS manifest for that rendition (the chunklist).
The output can be sent to files in localdisk or pushed using HTTP to any webserver you have configured as a live streaming origin.

This segmenter also implements "periscope" [LHLS](https://medium.com/@periscopecode/introducing-lhls-media-streaming-eb6212948bef) mode. This low latency mode is used in any output type (local files and HTTP).

This code is not designed to be used in any production workflow, it has been created as a learning GoLang exercise.

![Block diagram goes here](./pics/blockDiagramGoSegmenter.png "Block diagram")

# Usage
## Instalation
1. Just download GO in your computer. See [GoLang](https://golang.org/)
2. Create a Go directory to be used as a workspace for all go code, i.e.
```
mkdir ~/MYDIR/go
```
3. Add `GOPATH` to your `~/.bash_profile` or equivalent for your shell
```
export GOPATH="$HOME/MYDIR/go"
```
4. Add `GOPATH/bin` to your path in `~/.bash_profile` or equivalent for your shell
```
export PATH="$PATH:$GOPATH/bin
```
5. Install Glide. See [glide](https://github.com/Masterminds/glide)
6. Restart your terminal or source your profile
7. Clone this repo:
```
go get github.com/jordicenzano/go-ts-segmenter
```
8. Go the the source code dir `
```
cd $HOME/MYDIR/go/src/github.com/jordicenzano/go-ts-segmenter
```
9. Install the package dependencies:
```
glide up
```
10. Compile `main.go` doing:
```
make
```

## Testing
You can execute `./bin/./manifest-generator -h` to see all the possible command arguments.
```
Usage of ./bin/./manifest-generator:
  -apid int
        Audio PID to parse (default -1)
  -apids
        Enable auto PID detection, if true no need to pass vpid and apid (default true)
  -cf string
        Chunklist filename (default "chunklist.m3u8")
  -d int
        Indicates where the destination (0- No output, 1- File + flag indicator, 2- HTTP chunked transfer) (default 1)
  -f string
        Chunks base filename (default "chunk_")
  -host string
        HTTP Host (default "localhost:9094")
  -i int
        Indicates where to put the init data PAT and PMT packets (0- No ini data, 1- Init segment, 2- At the begining of each chunk (default 2)
  -l int
        If > 0 activates LHLS, and it indicates the number of advanced chunks to create
  -lf string
        Logs file (default "./logs/segmenter.log")
  -m int
        Manifest to generate (0- Vod, 1- Live event, 2- Live sliding window (default 2)
  -p string
        Output path (default "./results")
  -protocol string
        HTTP Scheme (http, https) (default "http")
  -t float
        Target chunk duration in seconds (default 4)
  -v    enable to get verbose logging
  -vpid int
        Video PID to parse (default -1)
  -w int
        Live window size in chunks (default 3)
```
## Examples
- Generate simple HLS from a test VOD TS file to my local disc in `./results/vod`:
```
cat ./fixture/testSmall.ts| ./bin/manifest-generator -p ./results/vod
```
- Generate simple HLS from a test **live** stream to my local disc in `./results/live` (requires [ffmpeg](https://ffmpeg.org/)):
```
ffmpeg -f lavfi -re -i smptebars=duration=6000:size=320x200:rate=30 -f lavfi -i sine=frequency=1000:duration=6000:sample_rate=48000 -pix_fmt yuv420p -c:v libx264 -b:v 180k -g 60 -keyint_min 60 -profile:v baseline -preset veryfast -c:a aac -b:a 96k -f mpegts - | bin/manifest-generator -p ./results/live
```

You can also add some overlays to indicate the frame number by doing:
```
ffmpeg -f lavfi -re -i smptebars=duration=6000:size=320x200:rate=30 -f lavfi -i sine=frequency=1000:duration=6000:sample_rate=48000 -pix_fmt yuv420p -c:v libx264 -b:v 180k -g 60 -keyint_min 60 -profile:v baseline -preset veryfast -c:a aac -b:a 96k -vf "drawtext=fontfile=/Library/Fonts/Arial.ttf: text=\'Local time %{localtime\: %Y\/%m\/%d %H.%M.%S} (%{n})\': x=10: y=10: fontsize=16: fontcolor=white: box=1: boxcolor=0x00000099" -f mpegts - | bin/manifest-generator -p ./results/ffmpeg -p ./results/live
```
Note: The previous snippet only works on MAC OS, you should probably remove (or modify) the `fontfile` path if you use another OS.

- Generate **LHLS** with 3 advanced chunks from a test **live** stream to my local disc in `./results/live` (requires [ffmpeg](https://ffmpeg.org/)):
```
ffmpeg -f lavfi -re -i smptebars=duration=6000:size=320x200:rate=30 -f lavfi -i sine=frequency=1000:duration=6000:sample_rate=48000 -pix_fmt yuv420p -c:v libx264 -b:v 180k -g 60 -keyint_min 60 -profile:v baseline -preset veryfast -c:a aac -b:a 96k -f mpegts - | bin/manifest-generator -p ./results/live -l 3
```
Note: To serve the LHLS data generated by this application you need to use [webserver-chunked-growingfiles](https://github.com/jordicenzano/webserver-chunked-growingfiles). The stream will play in any HLS compatible player, but if you really want t see ultra low latency you will need to use this player (TODO)

# TODO
- HTTP out not tested, test it
- Add examples for HTTP out and point to Matt's repo