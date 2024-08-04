#!/usr/bin/env roll

# Disableton Shell

## Command-Line Music Production Tool

Work of Faddy Michel

In Solidarity with The People of Palestine till Their Whole Land is FREE

## Prerequisites

* [Node.js](https://nodejs.org)
* [npm](https://npmjs.org)
* [Csound](https://csound.com)

## Installation

```sh
sudo npm i -g disableton
```

## Usage

```sh
disableton [ ... notation ]
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
?# cd .dsbltn/engine ; if [ ! -d node_modules/@faddys/scenarist ] ; then npm i @faddys/scenarist ; fi
```

### `.dsbltn/engine/node_modules/@faddys/command`

```roll
?# cd .dsbltn/engine ; if [ ! -d node_modules/@faddys/command ] ; then npm i @faddys/command ; fi
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
import { stdin as input, stdout as output } from 'node:process';import Disableton from './index.mjs';

const directory = await $0 ( '_dsbdir' )
.then ( $ => $ ( Symbol .for ( 'output' ) ) )
.then ( ( [ directory ] ) => directory );

const $ = await Scenarist ( new Disableton ( ... process .argv .slice ( 2 ) ) );

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

import Note from './note.mjs';
import File from './file.mjs';

export default class Disableton {

#argv

constructor ( ... argv ) { this .#argv = argv }

#player
#location

async $_producer ( $, { player, location } ) {

this .#player = player;
this .#location = location;

await $ ( Symbol .for ( 'list' ) );

if ( this .#argv .length )
await $ ( ... this .#argv );

if ( this .#player ) {

await this .#player ( Symbol .for ( 'write' ), 'dsbltn/' + location [ location .length - 1 ] );

return;

}

}

async $score ( $, ... argv ) {

if ( ! argv .length )
return;

const notation = await $0 ( 'cat', argv .shift () )
.then ( async $ => ( {

output: await $ ( Symbol .for ( 'output' ) ),
error: await $ ( Symbol .for ( 'error' ) )

} ) );

if ( notation .error .length )
throw notation .error .join ( '\n' );

for ( let line of notation .output )
if ( ( line = line .trim () ) .length )
await $ ( ... line .split ( /\s+/ ) );

}

async $write () {

const score = await $0 ( 'cat - > .dsbltn/index.sco' );

for ( const note of this )
await score ( typeof note === 'object' ? this .note ( note ) : note );

await score ( Symbol .for ( 'end' ) );

}

#engine

async $play () {

this .#engine = await $0 ( 'csound -odac .dsbltn/index.orc .dsbltn/sco' );

}

static tempo = 105
#tempo

async $tempo ( $, ... argv ) {

if ( ! argv .length )
return this .#tempo || ( this .#player ? await this .#player ( 'tempo' ) : Disableton .tempo );

this .#tempo = parseFloat ( argv .shift () );

if ( isNaN ( this .#tempo ) ) {

await $ ( Symbol .for ( 'write' ), 'tempo', this .#tempo = await this .#player ( 'tempo' ) );

throw `Tempo is set to a non-numeric value. Instead, it's reset to it's player tempo ${ this .#tempo }`;

}

$ ( Symbol .for ( 'write' ), 'tempo', this .#tempo );

return ! argv .length ? this .#tempo : $ ( ... argv );

}

$dsbltn = Disableton

$note = Note

$_director = new File

};

//-==
```

### `.dsbltn/engine/note.mjs`

```roll
?# cat - > .dsbltn/engine/note.mjs
```

```js
//+==

import File from './file.mjs';

export default class Note {

static instance = 0
#instance

constructor ( ... argv ) {

this .#instance = ++Note .instance % 10 === 0 ? ++Note .instance : Note .instance;

}

#player

async $_producer ( $, { player } ) {

this .#player = player;

await $ ( Symbol .for ( 'list' ) );

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

$_director = new File ( 'note' )

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

constructor ( type = 'dsbltn' ) { this .type = type }

async $_producer ( $, { player, location } ) {

this .player = player;

const recursive = true;

await make ( this .$directory = [ ... location .slice ( 0, -1 ), '.dsbltn/data/' ] .join ( '/' ), { recursive } );

if ( this .type !== 'dsbltn' )
return;

await make ( this .$directory + 'dsbltn', { recursive } );
await make ( this .$directory + 'note', { recursive } );

}

async $_list ( $ ) {

for ( const direction of await list ( this .$directory, { recursive: true } ) )
if ( ! [ 'dsbltn', 'note' ] .includes ( direction ) )
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
