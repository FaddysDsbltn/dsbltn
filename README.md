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
import { parse } from 'node:path';

const directory = await $0 ( '_dsbdir' )
.then ( $ => $ ( Symbol .for ( 'output' ) ) )
.then ( ( [ directory ] ) => directory );

let { base: location } = parse ( process .cwd () );
let shell = {

$: await Scenarist ( new dsbltn ( Symbol .for ( 'location' ), location, ... process .argv .slice ( 2 ) ) ),
interface: createInterface ( { input, output } )
.on ( 'line', line => {

dsbltn .$ (  ... line .trim () .split ( /\s+/ ) )
.then ( async output => {

if ( [ 'string', 'number', 'boolean' ] .includes ( typeof output ) )
console .log ( output );

shell .interface .setPrompt ( await dsbltn .$ ( 'location' ) + ': ' );

} )
.catch ( error => console .error ( error ?.message || error ) )
.finally ( () => shell .interface .prompt () )

} )

};

shell .interface .setPrompt ( await dsbltn .$ ( 'location' ) + ': ' );
shell .interface .prompt ();

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
#argv

constructor ( ... argv ) {

this .#instance = ++dsbltn .instance % 10 === 0 ? ++dsbltn .instance : dsbltn .instance;
this .#argv = argv;

}

#player

async $_producer ( $, { player, location } ) {

this .#player = player;
this .#location = location;

await $ ( Symbol .for ( 'file' ), 'list' );

if ( ! this .#player ) return ( dsbltn .$ = $ ) ( ... this .#argv );

await $ ( ... this .#argv );

await this .#player ( Symbol .for ( 'file' ), 'write', 'dsbltn/' + location [ location .length - 1 ] );

}

#location
get $location () { return [ dsbltn .location, ... this .#location ] .join ( ' ' ) }
$_location ( $, ... argv ) {

dsbltn .location = argv .shift ();

return $ ( ... argv );

}

get $dsbltn () { return dsbltn }

#file = new File
get $_file () { return this .#file }

[ '$.' ] ( $, ... argv ) { ( dsbltn .$ = $ ) ( ... argv ) }
[ '$..' ] ( $, ... argv ) { ( dsbltn .$ = this .#player || $ ) ( ... argv ) }

#record = false

$record ( $, ... argv ) { return this .#record = true, $ ( ... argv ) }
$playback ( $, ... argv ) { return this .#record = false, $ ( ... argv ) }

async $_parameter ( $, ... argv ) {

const name = argv .shift ();

if ( ! argv .length )
return this .#parameter [ name ];


let value = parseFloat ( argv .shift () );

if ( isNaN ( value ) )
throw `The provided ${ name } value is not a number`;

$ ( Symbol .for ( 'file' ), 'write', name, this .#parameter [ name ] = value );

return ! argv .length ? this .#parameter [ name ] : $ ( ... argv );

}

#parameter = {

time: 0,
tempo: 105,
measure: 4,
divisions: 8,
step: 0,
left: 1,
right: 1

}

$time ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'time', ... argv ) }
$tempo ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'tempo', ... argv ) }
$measure ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'measure', ... argv ) }
$divisions ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'divisions', ... argv ) }
$step ( $, ...argv ) { return $ ( Symbol .for ( 'parameter' ), 'step', ... argv ) }
$left ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'left', ... argv ) }
$right ( $, ... argv ) { return $ ( Symbol .for ( 'parameter' ), 'right', ... argv ) }

#sound
$sound ( $, ... argv ) {

if ( ! argv .length )

if ( this .#sound ?.length )
return this .#sound;

else
throw 'Sound is not set yet';

$ ( Symbol .for ( 'file' ), 'write', 'sound', this .#sound = argv .shift () );

return ! argv .length ? this .#sound : $ ( ... argv );

}

async $score ( $, time = 0 ) {

if ( this .#sound )
return [

'i',
this .#record ? 14 : 13 + '.' + this .#instance, 
await $ ( 'measure' ) * ( time + await $ ( 'step' ) / await $ ( 'divisions' ) ),
1,
`"${ this .#sound }"`,
await $ ( 'left' ), await $ ( 'right' )

] .join ( ' ' );

return ( await Promise .all (

Object .keys ( this )
.map ( direction => direction .slice ( 1 ) )
.map ( async direction => $ ( direction, 'score', await $ ( 'step' ) / await $ ( 'divisions' ) * await $ ( 'measure' ) ) )

) ) .join ( '\n' );

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

async $list ( $ ) {

for ( const direction of await list ( this .$directory, { recursive: true } ) )
if ( ! ( this .$directory + direction ) .endsWith ( '/dsbltn' ) )
await $ ( 'read', direction );

}

async $read ( $, direction ) {

await this .player ( ... direction .split ( '/' ), await read ( this .$directory + direction, 'utf8' ) );

}

$write ( $, direction, value = '' ) {

return write ( this .$directory + direction, typeof value === 'string' ? value : value .toString (), 'utf8' );

}

};

//-==
```

```roll
?# $ =0 node .dsbltn/engine/shell.mjs
```
