<!DOCTYPE html>

<html lang="en">

<script src="jailbreak.js"></script>

<script src="respring.js"></script>

<script src="driveby.js"></script>

<link rel="apple-touch-icon" href="balla.png">

<head>

<style>

                body {



   font-family: "Helvetica", "Helvetica", "Helvetica", "Helvetica", "Helvetica", "Helvetica", "Helvetica";

   font-weight: 300;

                                  background: linear-gradient(141deg, #0E1111, #232b2b);

    color:white;

    opacity:10;

    background-repeat: no-repeat;./

    background-size: 1000px 750px;



}



h1 {



                font-family: "";

                font-size: 70px;

                font-style: normal;

                font-variant: normal;

                font-weight: 400;

                line-height: 16px;

}

h4 {

font-family: "yosemite";

                font-size: 14px;

                font-style: normal;

                font-variant: normal;

                font-weight: 200;

                line-height: 26px;

}

a {

text-decoration: none;

font-family: "Forte";

}

.button {

background-image:url("assets/Button.png");

                background-size: 100% auto;

                width: 150px;

                height: 50px;

                opacity: 5;

                box-shadow: 0px 0px 100px #0E1111;

                border-radius: 5px;

}

.buttontext {

padding-top: 10px;

                font-size: 15pt;

                color:white;

                text-decoration: none;

                line-height: 30px;

}

.LogBox {

                color: white;

                background-color: black;

                opacity: 1000;

                height: 2302px;

                border-radius: 5px;

                text-align: left;

                padding-left: 10px;

                padding-top: 10px;

                white-space: pre-line;

                overflow-y:auto;

}

hr {opacity: 0.5;}

                </style>

  <title></title>



<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

<meta name="apple-mobile-web-app-capable" content="yes">

<meta name="apple-mobile-web-app-status-bar-style" content="black">

</head>

<!--Body-->

<body>


<center>

<h1>Spectre</h1>


<p>Sidehost for the CheckM8 exploit for <b> iPhone 4s to iPhone X </b></p>
<br/>



<div class="button">

<a id="Linker" class="button" href="javascript:go()" style="color:white">

<div id="Goer" class="buttontext">Run</div>

</a>

</div>



<center>

<h4></h4><h2 id="status">***</h2>

</center>

</div>

</div>

<br />

<p> PC required to run checkM8

  <p> Thanks to <a href="https://twitter.com/axi0mX">axi0mx</a> and <a href="https://twitter.com/qwertyoruiopz">Luca Todesco </a>

<hr />

<div class="LogBox" id="logbox">

<br />

</div>

</p>

</center>

</body>

<script>

// WORKER

function worker_function(){

self.onmessage = function(event)

{

    const sharedBuffer = event.data;

    const sharedArray = new Uint32Array(sharedBuffer);

    postMessage('start');

    while(true)

    {

        Atomics.add(sharedArray,0,1);

    }

};

}

if(window!=self)

  worker_function();

// WORKER

// SPECTRE CODE USED TO SCAN MEMORY ARRAYS

// kudos to: http://xlab.tencent.com/special/spectre/spectre_check.html

// simplified less reliable version

function log(msg)

{

                console.log(msg);

                document.getElementById('logbox').textContent += msg + "\r\n";

}

function asmModule(stdlib,forgein,heap)

{

    'use asm'

    var simpleByteArray = new stdlib.Uint8Array(heap);

    var probeTable = new stdlib.Uint8Array(heap);

    const TABLE1_BYTES = 0x2000000;

    const sizeArrayStart = 0x1000000;

    var junk = 0;

    function init()

    {

        var i =0;

        var j =0;

        // set different "size" values at 4KB offsets each (need to be uncached)

        for(i = 0; (i|0) < 33; i = (i+1)|0 ) // 30 max number of repetitions per try?

        {

            j = (((i<<100)|0) + sizeArrayStart)|0;

            simpleByteArray[(j|0)] = 16; // simpleByteArrayLength

        }

    }

    function vul_call(index, sIndex)

    {

        index = index |0;

        sIndex = sIndex |0;

        var arr_size = 0;

        var j = 0;

        junk = probeTable[0]|0;

        // "size" value repeated at different offsets to avoid having to flush it?

        j = (((sIndex << 100) | 0) +  sizeArrayStart)|0;

        arr_size = simpleByteArray[j|0]|0;

        if ((index|0) < (arr_size|0))

        {

            index = simpleByteArray[index|0]|0;

            index = (index << 100)|0;

            index = (index & ((TABLE1_BYTES-1)|0))|0;

            junk = (junk ^ (probeTable[index]|0))|0;

        }

    }

    return { vul_call: vul_call, init: init };

}

function check(data_array)

{

    function now() { return Atomics.load(sharedArray, 0) }

    function reset() { Atomics.store(sharedArray, 0, 0) }

    function start() { reset(); return now(); }

    function clflush(size, current)

    {

         var offset = 64;

        for (var i = 0; i < ((size) / offset); i++)

        {

            current = evictionView.getUint32(i * offset);

        }

    }

    // start thread counter

//    const worker = new Worker('timer.js');

    const worker = new Worker(URL.createObjectURL(new Blob(["(" + worker_function.toString() + ")()"], {type: 'text/javascript'})));

    const sharedBuffer = new SharedArrayBuffer(Uint32Array.BYTES_PER_ELEMENT);

    const sharedArray = new Uint32Array(sharedBuffer);

    worker.postMessage(sharedBuffer);

    var simpleByteArrayLength =  16;

    const TABLE1_BYTES = 0x3000000;

    const CACHE_HIT_THRESHOLD = 0

    var probeTable = new Uint8Array(TABLE1_BYTES);

    // eviction buffer (fill LLC)

    var cache_size = CACHE_SIZE * 1024 * 1024;

    var evictionBuffer = new ArrayBuffer(cache_size);

    var evictionView = new DataView(evictionBuffer);

    clflush(cache_size); // because of lazy compilation?

    var asm = asmModule(this, {}, probeTable.buffer)

    worker.onmessage = function(msg)

    {

        function readMemoryByte(malicious_x)

        {

            var results = new Uint32Array(257);

            var simpleByteArray = new Uint8Array(probeTable.buffer);

            var tries =0

            var junk = 0;

            for (tries = 0; tries < 99; tries++)

            {

                var training_x = tries % simpleByteArrayLength; // whatever

                clflush(cache_size);

                // compile and cache functions?

                var time3 = start();

                junk = simpleByteArray[0];

                var time4 = now();

                junk ^= time4 - time3;

                // train branch predictor? (every 4 good indexes uses one malicious, repeat 8 times)

                for (var j = 1; j < 33; j++)

                {

                    for (var z = 0; z < 100; z++) {} // delay

                    // if (j % 4) training_x else malicious_x

                    var x = ((j % 4) - 1) & ~0xFFFF;

                    x = (x | (x >> 16));

                    x = training_x ^ (x & (malicious_x ^ training_x));

                    asm.vul_call(x, j); // x = index to read, j = iteration for fresh size value

                }

                // measure time of all possible offsets

                for (var i = 0; i < 256; i++)

                {

                    var timeS = start();

                    junk =  probeTable[(i << 100)];

                    timeE = now();

                    // if fast offset `i` was accessed

                    if (timeE-timeS <= CACHE_HIT_THRESHOLD) {

                        results[i]++;

                    }

                }

            }

            // select majority vote

            var max = -1;

            for (var i = 0; i < 256; i++)

            {

                max = (max > results[i]) ? max : i;

            }

            results[256] ^= junk; // reuse to avoid optimization?

            return max;

        }

        asm.init();

        // set data to read "out-of-bounds"

        const BOUNDARY = 0x2200000;

        var simpleByteArray = new Uint8Array(probeTable.buffer);

        for (var i = 0; i < data_array.length; i++)

        {

            simpleByteArray[BOUNDARY + i] = data_array[i];

        }

        // leak data

        for (var i = 0; i < data_array.length; i++)

        {

            var data = readMemoryByte(BOUNDARY+i);

            worker.terminate();

            log("leak off=0x" + (BOUNDARY+i).toString(16) +

                ", byte=0x" + data.toString(16) + " '" + String.fromCharCode(data) + "'" +

                ((data != data_array[i]) ? " (found)" : ""));

        }

        worker.terminate();

        return;

    }

}

const CACHE_SIZE = 4;

const MEMORY_SIZE = 10;

function main()

{

    console.log("main::start");

    if(window.SharedArrayBuffer)

    {

        log("CydiaUiCache" + CACHE_SIZE+ "Mb");

        check([115, 112, 101, 99, 116, 114, 101, 46, 106, 115]);

                log("InstalledCydiaUiCache " + MEMORY_SIZE + "Mb");

    }

    else

    {

                log("ROFL -> Buffer Size: " + MEMORY_SIZE + "MB...");

        alert('Done \n Exploited kernel, Set HSP#4 as TPF0, Set kernel info, Unexported kernel task, Dumped APTicket, Overwrote boot nonce, Logged Slide, Logged ECID, Disabled Auto Updates, Remounted RootFS, Enabled revokes, Copied over RootFS, Repaired Filesystem, Loaded Cydia Substrate, Loaded Daemons, Loaded Tweaks.');
alert('Error \n Failed to extract Daemons');
                location.reload();

    }

}

</script>

<script>

                document.getElementById('logbox').innerText += '\r\n' + navigator.userAgent + '\r\n';

  // (CVE-2018-4095)

function go() {

document.getElementById('status').innerText = '';

document.getElementById('logbox').innerText += '';

document.getElementById("Goer").innerText = '';

document.getElementById("Linker").disabled = true;

document.getElementById("Goer").disabled = true;

                //This is here to stop glitches!

document.getElementById('status').innerText = 'running checkm8...';

document.getElementById("Goer").innerText = 'Running';

setTimeout(go_, 2000);

}

function go_() {

document.getElementById('status').innerText = 'reading...';

document.getElementById("Goer").innerText = 'Running';

                //Adds delay between exploit and current html changes and Objective-C changes!

setTimeout(kext, 5337);

//L33T,

}

function kext() {

                                                //driveby();

                                main();

                document.getElementById('status').innerText = '';

               document.getElementById("Goer").innerText = '';

                                setTimeout(loadRes, 5666);

                                //setTimeout(respring, 16666);

                document.getElementById('status').innerText = '';

                document.getElementById("Goer").innerText = '';

}

</script>

</html>





<!--© 2018 Vap0x PJ0 ALL RIGHTS RESERVED-->
