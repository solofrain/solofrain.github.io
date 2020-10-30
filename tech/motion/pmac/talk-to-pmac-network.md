# Talking to Turbo PMAC over Network

## Tcl

You'll need [TCP and recv](https://wiki.tcl-lang.org/page/TCP+and+recv).

You'll also need to review the PMAC Ethernet protocol documentation, this library is a pretty direct mapping of the available commands. The only "fancy" usage here is pmac:command which provides a command reply pair and pmac:mapmem which reads a memory buffer from the pmac and immediately calls [binary](https://wiki.tcl-lang.org/page/binary) scan to map the data buffer into Tcl variables that need to be fully qualified names.

```tcl
set Host2PMAC          0x40
set PMAC2Host          0xC0

set PMAC_SENDLINE      0xB0
set PMAC_GETLINE       0xB1
set PMAC_FLUSH         0xB3
set PMAC_GETMEM        0xB4
set PMAC_SETMEM        0xB5
set PMAC_READY         0xC2
set PMAC_GETBUFFER     0xC5
set PMAC_WRITEBUFFER   0xC6

proc pmac:connect { pmac } {
    set sock [socket $pmac 1025]
    fconfigure $sock -encoding binary

    return $sock
}

proc pmac:flush { sock } {
    pmac:packet $sock $::Host2PMAC $::PMAC_FLUSH
    read $sock 1
}

proc pmac:wait { sock } {
    set x 0
    while { 1 } {
        pmac:packet $sock $::PMAC2Host $::PMAC_READY 0 2
        binary scan [read   $sock 2] c x

        if { $x } { break }
        after 10
    }
}

proc pmac:getbuf { sock } {
    pmac:packet $sock $::PMAC2Host $::PMAC_GETBUFFER 0 1400
    recv $sock
}


proc pmac:command { sock data } {
    pmac:packet $sock $::Host2PMAC $::PMAC_SENDLINE  0 [string length $data] $data
    binary scan [read $sock 1] c reply

    pmac:wait   $sock
    string map { \r " " } [string range [pmac:getbuf $sock] 0 end-2]
}

proc pmac:wbuff { sock data here } {
    pmac:packet $sock $::Host2PMAC $::PMAC_WRITEBUFFER  0 [string length $data] $data
    binary scan [read $sock 4] scc line code error

    if { $error == 0x80 } {
        error "pmac:write : error $code at line [expr $here + $line]"
    }
}

proc pmac:write { sock Data } {
    set data {}
    set Here 0
    set here 0
    foreach line [split $Data \n] {
        if { [string length $data] + [string length $line] + 1 > 1000 } {
            pmac:wbuff $sock $data $Here
            incr   Here $here
            set data {}
            set here 0
        }

        append data $line "\0"
    }
    pmac:wbuff $sock $data $Here
}

proc pmac:setmem { sock start length } {
    pmac:packet $sock $::Host2PMAC $::PMAC_SETMEM $start $offset $data
}

proc pmac:packet  { sock type requ { offset 0 } { size 0 } { data {} } } {
    puts -nonewline $sock [binary format ccSSS $type $requ $offset 0 $size]
    puts -nonewline $sock $data
    flush $sock
}

proc pmac:getmem { sock start length } {
    pmac:packet $sock $::PMAC2Host $::PMAC_GETMEM $start $length
    recv   $sock
}

proc pmac:mapmem { sock start length scan vars } {
    set block  [pmac:getmem $sock $start $length]
    binary scan $block $scan {*}$vars]
}
```