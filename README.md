# node-uuid-v4
Very fast node-only uuid v4 (random) generator.

This library is not intended to be published to npm instead just copy it into
your code.

# Usage

```javascript
const uuid = require('./uuid.js')();

const id = uuid();

```

# Background
When using unique ID:s generated using https://github.com/broofa/node-uuid I
decided that I might as well pull in the code for the uuid v4 generation instead
of having a dependency.

After a cleanup and striping out everything not related to `nodejs`:

```javascript
const hex = [];
for (let i = 0; i < 256; i++) hex[i] = (i + 0x100).toString(16).substr(1);

function v4() {
  var r = crypto.randomBytes(16);
  r[6] = (r[6] & 0x0f) | 0x40;
  r[8] = (r[8] & 0x3f) | 0x80;
  return  hex[r[0x0]] + hex[r[0x1]] +
          hex[r[0x2]] + hex[r[0x3]] + '-' +
          hex[r[0x4]] + hex[r[0x5]] + '-' +
          hex[r[0x6]] + hex[r[0x7]] + '-' +
          hex[r[0x8]] + hex[r[0x9]] + '-' +
          hex[r[0xa]] + hex[r[0xb]] +
          hex[r[0xc]] + hex[r[0xd]] +
          hex[r[0xe]] + hex[r[0xf]];
}
```

Well you can't look at that and not wonder if you can make it smaller if you are
a programmer. Right? :)

But unfortunately someone beat me to it...

https://gist.github.com/jed/982883

Apparently that solution is slow so feeling pumped again I decided to make it
faster. Caffeine for the optimize God!

Using the benchmark from node-uuid I measured the above snippet to produce
about 415k uuids per second.

Trying different things like working on 16 bits instead of 8 using Uint16Array
only slowed things down, but suprisingly a change in how the hex table was
created did give some improvements.

```javascript
const hex = [
  '00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '0a', '0b', '0c',
  '0d', '0e', '0f', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19',
  '1a', '1b', '1c', '1d', '1e', '1f', '20', '21', '22', '23', '24', '25', '26',
  '27', '28', '29', '2a', '2b', '2c', '2d', '2e', '2f', '30', '31', '32', '33',
  '34', '35', '36', '37', '38', '39', '3a', '3b', '3c', '3d', '3e', '3f', '40',
  '41', '42', '43', '44', '45', '46', '47', '48', '49', '4a', '4b', '4c', '4d',
  '4e', '4f', '50', '51', '52', '53', '54', '55', '56', '57', '58', '59', '5a',
  '5b', '5c', '5d', '5e', '5f', '60', '61', '62', '63', '64', '65', '66', '67',
  '68', '69', '6a', '6b', '6c', '6d', '6e', '6f', '70', '71', '72', '73', '74',
  '75', '76', '77', '78', '79', '7a', '7b', '7c', '7d', '7e', '7f', '80', '81',
  '82', '83', '84', '85', '86', '87', '88', '89', '8a', '8b', '8c', '8d', '8e',
  '8f', '90', '91', '92', '93', '94', '95', '96', '97', '98', '99', '9a', '9b',
  '9c', '9d', '9e', '9f', 'a0', 'a1', 'a2', 'a3', 'a4', 'a5', 'a6', 'a7', 'a8',
  'a9', 'aa', 'ab', 'ac', 'ad', 'ae', 'af', 'b0', 'b1', 'b2', 'b3', 'b4', 'b5',
  'b6', 'b7', 'b8', 'b9', 'ba', 'bb', 'bc', 'bd', 'be', 'bf', 'c0', 'c1', 'c2',
  'c3', 'c4', 'c5', 'c6', 'c7', 'c8', 'c9', 'ca', 'cb', 'cc', 'cd', 'ce', 'cf',
  'd0', 'd1', 'd2', 'd3', 'd4', 'd5', 'd6', 'd7', 'd8', 'd9', 'da', 'db', 'dc',
  'dd', 'de', 'df', 'e0', 'e1', 'e2', 'e3', 'e4', 'e5', 'e6', 'e7', 'e8', 'e9',
  'ea', 'eb', 'ec', 'ed', 'ee', 'ef', 'f0', 'f1', 'f2', 'f3', 'f4', 'f5', 'f6',
  'f7', 'f8', 'f9', 'fa', 'fb', 'fc', 'fd', 'fe', 'ff'
];
````

Now we get about 460k uuids per second. This is probably something that depends
on the internals of v8 and might not be a stable optimisation on other platforms.

Now for the final boost: Allocating random numbers in larger chunks gives a
massive optimisation!

```javascript
const n = 768;
let b, i = n;

const rnd = () => {
  if (i >= n) {
    b = crypto.randomBytes(n);
    i = 0;
  }
  return b.slice(i, i += 16);
};

function v4() {
  const r = rnd();
  r[6]  = (r[6] & 0x0f) | 0x40;
  r[8]  = (r[8] & 0x3f) | 0x80;
  return hex[r[0x0]] + hex[r[0x1]] +
         hex[r[0x2]] + hex[r[0x3]] + '-' +
         hex[r[0x4]] + hex[r[0x5]] + '-' +
         hex[r[0x6]] + hex[r[0x7]] + '-' +
         hex[r[0x8]] + hex[r[0x9]] + '-' +
         hex[r[0xa]] + hex[r[0xb]] +
         hex[r[0xc]] + hex[r[0xd]] +
         hex[r[0xe]] + hex[r[0xf]];
}
```

This final optimisation gives about 2M uuids per second. That's more than a 4x
increase from the original code. Nice!







