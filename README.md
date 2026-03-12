# tcl-http-parser

TCL Http Parser lib based on Pico HTTP Parser [https://github.com/h2o/picohttpparser]


### Install

```bash
$ mkdir build && cd build && cmake ..
$ make
$ sudo make install
```

### Usage

```tcl

proc http_handle {socket addr port} {
  
package require httpparser

proc http_handle {socket addr port} {

  puts http_handle

  set req [::httpparser::parse -channel $socket]

  puts "::> method  = [dict get $req method]"
  puts "::> path  = [dict get $req path]"
  puts "::> path_raw  = [dict get $req path_raw]"
  puts "::> frag  = [dict get $req frag]"
  puts "::> body  = [dict get $req body]"
  puts "::> headers = [dict get $req headers]"
  puts "::> queries = [dict get $req queries]"

  puts $socket "HTTP/1.1 200 OK\nConnection: close\nContent-Type: text/plain\n"
  close $socket
}

proc accept {socket addr port} {
  chan configure $socket -blocking 0 -buffering line
  chan event $socket readable [list http_handle $socket $addr $port]
}

socket -server accept 5151

puts "server run on http://localhost:5151"

vwait forever

```
