#!/usr/bin/env roll

# dsbltn Shell

## Command-Line Music Production Tool

Work of Faddy Michel

In Solidarity with The People of Palestine till Their Whole Land is FREE

## Prerequisites

* [Node.js](https://nodejs.org)
* [npm](https://npmjs.org)
* [Csound](https://csound.com)

## Installation

```sh
sudo npm i -g dsbltn
```

## Usage

```sh
dsbltn [ ... notation ]
```

## `.dsbltn/engine`

```roll
?# mkdir -p .dsbltn/engine
```

### `.dsbltn/engine/index.orc`

```roll
?# cd .dsbltn/engine ; if [ ! -f index.orc ] ; then cat - > index.orc ; fi
```

```csound
//+==

sr = 48000
ksmps = 32
nchnls = 6
0dbfs = 1

instr 13, 14, beat

p3 *= 1000
SPath strget p4
strset p5, SPath

iEvent init ( int ( p1 ) % 10 ) + frac ( p1 )

kLoop metro 1/p3

if kLoop == 1 then

schedulek iEvent, 0, 1, p5, p6, p7

endif

endin

instr 3, playback

SPath strget p4
p3 filelen SPath

aLeft, aRight diskin2 SPath

outs aLeft / ( p5 + 1 ), aRight / ( p6 + 1 )

endin

instr 4, recorder

SPath strget p4

SPath1 strcat SPath, ".1.2.wav"
SPath2 strcat SPath, ".3.4.wav"
SPath3 strcat SPath, ".5.6.wav"

aLeft1, aRight1 inch 1, 2
aLeft2, aRight2 inch 3, 4
aLeft3, aRight3 inch 5, 6

fout SPath1, -1, aLeft1, aRight1
fout SPath2, -1, aLeft2, aRight2
fout SPath3, -1, aLeft3, aRight3

endin

//-==
```

### `.dsbltn/engine/node_modules/@faddys/scenarist`

```roll
?# -1 -2 cd .dsbltn/engine ; if [ ! -d node_modules/@faddys/scenarist ] ; then npm i @faddys/scenarist ; fi
```

### `.dsbltn/engine/node_modules/@faddys/command`

```roll
?# -1 -2 cd .dsbltn/engine ; if [ ! -d node_modules/@faddys/command ] ; then npm i @faddys/command ; fi
```

### `.dsbltn/engine/shell.mjs`

```roll
?# cat - > .dsbltn/engine/shell.mjs
```

```js
//+==

import Scenarist from '@faddys/scenarist';
import $0 from '@faddys/command';
import { createInterface } from 'node:readline';
import { stdin as input, stdout as output } from 'node:process';import dsbltn from './index.mjs';

const directory = await $0 ( '_dsbdir' )
.then ( $ => $ ( Symbol .for ( 'output' ) ) )
.then ( ( [ directory ] ) => directory );

const $ = await Scenarist ( new dsbltn ( ... process .argv .slice ( 2 ) ) );

createInterface ( { input, output } )
.on ( 'line', function process ( line ) {

$ (  ... line .trim () .split ( /\s+/ ) )
.then ( output => ( [ 'string', 'number', 'boolean' ] .includes ( typeof output ) ? console .log ( output ) : undefined ) )
.catch ( error => console .error ( error ?.message || error ) )
.finally ( () => this .prompt () )

} ) .prompt ();

//-==
```

### `.dsbltn/engine/index.mjs`

```roll
?# cat - > .dsbltn/engine/index.mjs
```

```js
//+==

import File from './file.mjs';

export default class dsbltn {

static instance = 0
#instance

constructor () { this .#instance = ++dsbltn .instance % 10 === 0 ? ++dsbltn .instance : dsbltn .instance }

#player
#location

async $_producer ( $, { player, location } ) {

this .#player = player;
this .#location = location;

await $ ( Symbol .for ( 'list' ) );

if ( ! this .#player ) return;

await this .#player ( Symbol .for ( 'write' ), 'dsbltn/' + location [ location .length - 1 ] );

}

$dsbltn = dsbltn

$_director = new File

static tempo = 105
#tempo

async $tempo ( $, ... argv ) {

if ( ! argv .length )
return this .#tempo || ( this .#tempo = this .#player ? await this .#player ( 'tempo' ) : dsbltn .tempo );

const tempo = parseFloat ( argv .shift () );

if ( isNaN ( tempo ) )
throw 'The provided tempo value is not a number';

$ ( Symbol .for ( 'write' ), 'tempo', this .#tempo = tempo );

return ! argv .length ? this .#tempo : $ ( ... argv );

}

#record = false

$record ( $, ... argv ) { return this .#record = true, $ ( ... argv ) }
$playback ( $, ... argv ) { return this .#record = false, $ ( ... argv ) }

async $score ( $ ) {

return `i ${ this .#record ? 14 : 13 }.${ this .#instance } `;
// ${ await $ ( 'time' ) } 1 "${ this .#path }" ${ this .#left } ${ this .#right }`;

}

#step = 0

async $step ( $, ... argv ) {

if ( ! argv .length )
return this .#step;

const step = parseFloat ( argv .shift () );

if ( isNaN ( step ) )
throw 'The provided note step value is not a number';

await $ ( Symbol .for ( 'write' ), 'step', this .#step =step );

if ( ! argv .length )
return this .#step;

return $ ( ... argv );

}

};

//-==
```

### `.dsbltn/engine/file.mjs`

```roll
?# cat - > .dsbltn/engine/file.mjs
```

```js
//+==

import {

mkdir as make,
readdir as list,
readFile as read,
writeFile as write

} from 'node:fs/promises';

export default class File {

async $_producer ( $, { player, location } ) {

this .player = player;
this .$directory = [ ... location .slice ( 0, -1 ), '.dsbltn/data/' ] .join ( '/' );

await make ( this .$directory + 'dsbltn', { recursive: true } );

}

async $_list ( $ ) {

for ( const direction of await list ( this .$directory, { recursive: true } ) )
if ( ! ( this .$directory + direction ) .endsWith ( '/dsbltn' ) )
$ ( Symbol .for ( 'read' ), direction );

}

async $_read ( $, direction ) {

await this .player ( ... direction .split ( '/' ), await read ( this .$directory + direction, 'utf8' ) );

}

$_write ( $, direction, value = '' ) {

return write ( this .$directory + direction, typeof value === 'string' ? value : value .toString (), 'utf8' );

}

};

//-==
```

```roll
?# $ =0 node .dsbltn/engine/shell.mjs
```
