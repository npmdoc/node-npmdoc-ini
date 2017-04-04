# api documentation for  [ini (v1.3.4)](https://github.com/isaacs/ini#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-ini.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-ini) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-ini.svg)](https://travis-ci.org/npmdoc/node-npmdoc-ini)
#### An ini encoder/decoder for node

[![NPM](https://nodei.co/npm/ini.png?downloads=true)](https://www.npmjs.com/package/ini)

[![apidoc](https://npmdoc.github.io/node-npmdoc-ini/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-ini_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-ini/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-ini/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-ini/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Isaac Z. Schlueter",
        "email": "i@izs.me",
        "url": "http://blog.izs.me/"
    },
    "bugs": {
        "url": "https://github.com/isaacs/ini/issues"
    },
    "dependencies": {},
    "description": "An ini encoder/decoder for node",
    "devDependencies": {
        "tap": "^1.2.0"
    },
    "directories": {},
    "dist": {
        "shasum": "0537cb79daf59b59a1a517dff706c86ec039162e",
        "tarball": "https://registry.npmjs.org/ini/-/ini-1.3.4.tgz"
    },
    "engines": {
        "node": "*"
    },
    "files": [
        "ini.js"
    ],
    "gitHead": "4a3001abc4c608e51add9f1d2b2cadf02b8e6dea",
    "homepage": "https://github.com/isaacs/ini#readme",
    "license": "ISC",
    "main": "ini.js",
    "maintainers": [
        {
            "name": "isaacs",
            "email": "i@izs.me"
        }
    ],
    "name": "ini",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/isaacs/ini.git"
    },
    "scripts": {
        "test": "tap test/*.js"
    },
    "version": "1.3.4"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module ini](#apidoc.module.ini)
1.  [function <span class="apidocSignatureSpan">ini.</span>decode (str)](#apidoc.element.ini.decode)
1.  [function <span class="apidocSignatureSpan">ini.</span>encode (obj, opt)](#apidoc.element.ini.encode)
1.  [function <span class="apidocSignatureSpan">ini.</span>parse (str)](#apidoc.element.ini.parse)
1.  [function <span class="apidocSignatureSpan">ini.</span>safe (val)](#apidoc.element.ini.safe)
1.  [function <span class="apidocSignatureSpan">ini.</span>stringify (obj, opt)](#apidoc.element.ini.stringify)
1.  [function <span class="apidocSignatureSpan">ini.</span>unsafe (val, doUnesc)](#apidoc.element.ini.unsafe)



# <a name="apidoc.module.ini"></a>[module ini](#apidoc.module.ini)

#### <a name="apidoc.element.ini.decode"></a>[function <span class="apidocSignatureSpan">ini.</span>decode (str)](#apidoc.element.ini.decode)
- description and source-code
```javascript
function decode(str) {
  var out = {}
    , p = out
    , section = null
    , state = "START"
           // section     |key = value
    , re = /^\[([^\]]*)\]$|^([^=]+)(=(.*))?$/i
    , lines = str.split(/[\r\n]+/g)
    , section = null

  lines.forEach(function (line, _, __) {
    if (!line || line.match(/^\s*[;#]/)) return
    var match = line.match(re)
    if (!match) return
    if (match[1] !== undefined) {
      section = unsafe(match[1])
      p = out[section] = out[section] || {}
      return
    }
    var key = unsafe(match[2])
      , value = match[3] ? unsafe((match[4] || "")) : true
    switch (value) {
      case 'true':
      case 'false':
      case 'null': value = JSON.parse(value)
    }

    // Convert keys with '[]' suffix to an array
    if (key.length > 2 && key.slice(-2) === "[]") {
        key = key.substring(0, key.length - 2)
        if (!p[key]) {
          p[key] = []
        }
        else if (!Array.isArray(p[key])) {
          p[key] = [p[key]]
        }
    }

    // safeguard against resetting a previously defined
    // array by accidentally forgetting the brackets
    if (Array.isArray(p[key])) {
      p[key].push(value)
    }
    else {
      p[key] = value
    }
  })

  // {a:{y:1},"a.b":{x:2}} --> {a:{y:1,b:{x:2}}}
  // use a filter to return the keys that have to be deleted.
  Object.keys(out).filter(function (k, _, __) {
    if (!out[k] || typeof out[k] !== "object" || Array.isArray(out[k])) return false
    // see if the parent section is also an object.
    // if so, add it to that, and mark this one for deletion
    var parts = dotSplit(k)
      , p = out
      , l = parts.pop()
      , nl = l.replace(/\\\./g, '.')
    parts.forEach(function (part, _, __) {
      if (!p[part] || typeof p[part] !== "object") p[part] = {}
      p = p[part]
    })
    if (p === out && nl === l) return false
    p[nl] = out[k]
    return true
  }).forEach(function (del, _, __) {
    delete out[del]
  })

  return out
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.ini.encode"></a>[function <span class="apidocSignatureSpan">ini.</span>encode (obj, opt)](#apidoc.element.ini.encode)
- description and source-code
```javascript
function encode(obj, opt) {
  var children = []
    , out = ""

  if (typeof opt === "string") {
    opt = {
      section: opt,
      whitespace: false
    }
  } else {
    opt = opt || {}
    opt.whitespace = opt.whitespace === true
  }

  var separator = opt.whitespace ? " = " : "="

  Object.keys(obj).forEach(function (k, _, __) {
    var val = obj[k]
    if (val && Array.isArray(val)) {
        val.forEach(function(item) {
            out += safe(k + "[]") + separator + safe(item) + "\n"
        })
    }
    else if (val && typeof val === "object") {
      children.push(k)
    } else {
      out += safe(k) + separator + safe(val) + eol
    }
  })

  if (opt.section && out.length) {
    out = "[" + safe(opt.section) + "]" + eol + out
  }

  children.forEach(function (k, _, __) {
    var nk = dotSplit(k).join('\\.')
    var section = (opt.section ? opt.section + "." : "") + nk
    var child = encode(obj[k], {
      section: section,
      whitespace: opt.whitespace
    })
    if (out.length && child.length) {
      out += eol
    }
    out += child
  })

  return out
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.ini.parse"></a>[function <span class="apidocSignatureSpan">ini.</span>parse (str)](#apidoc.element.ini.parse)
- description and source-code
```javascript
function decode(str) {
  var out = {}
    , p = out
    , section = null
    , state = "START"
           // section     |key = value
    , re = /^\[([^\]]*)\]$|^([^=]+)(=(.*))?$/i
    , lines = str.split(/[\r\n]+/g)
    , section = null

  lines.forEach(function (line, _, __) {
    if (!line || line.match(/^\s*[;#]/)) return
    var match = line.match(re)
    if (!match) return
    if (match[1] !== undefined) {
      section = unsafe(match[1])
      p = out[section] = out[section] || {}
      return
    }
    var key = unsafe(match[2])
      , value = match[3] ? unsafe((match[4] || "")) : true
    switch (value) {
      case 'true':
      case 'false':
      case 'null': value = JSON.parse(value)
    }

    // Convert keys with '[]' suffix to an array
    if (key.length > 2 && key.slice(-2) === "[]") {
        key = key.substring(0, key.length - 2)
        if (!p[key]) {
          p[key] = []
        }
        else if (!Array.isArray(p[key])) {
          p[key] = [p[key]]
        }
    }

    // safeguard against resetting a previously defined
    // array by accidentally forgetting the brackets
    if (Array.isArray(p[key])) {
      p[key].push(value)
    }
    else {
      p[key] = value
    }
  })

  // {a:{y:1},"a.b":{x:2}} --> {a:{y:1,b:{x:2}}}
  // use a filter to return the keys that have to be deleted.
  Object.keys(out).filter(function (k, _, __) {
    if (!out[k] || typeof out[k] !== "object" || Array.isArray(out[k])) return false
    // see if the parent section is also an object.
    // if so, add it to that, and mark this one for deletion
    var parts = dotSplit(k)
      , p = out
      , l = parts.pop()
      , nl = l.replace(/\\\./g, '.')
    parts.forEach(function (part, _, __) {
      if (!p[part] || typeof p[part] !== "object") p[part] = {}
      p = p[part]
    })
    if (p === out && nl === l) return false
    p[nl] = out[k]
    return true
  }).forEach(function (del, _, __) {
    delete out[del]
  })

  return out
}
```
- example usage
```shell
...
array[] = third value

You can read, manipulate and write the ini-file like so:

var fs = require('fs')
  , ini = require('ini')

var config = ini.parse(fs.readFileSync('./config.ini', 'utf-8'))

config.scope = 'local'
config.database.database = 'use_another_database'
config.paths.default.tmpdir = '/tmp'
delete config.paths.default.datadir
config.paths.default.array.push('fourth value')
...
```

#### <a name="apidoc.element.ini.safe"></a>[function <span class="apidocSignatureSpan">ini.</span>safe (val)](#apidoc.element.ini.safe)
- description and source-code
```javascript
function safe(val) {
  return ( typeof val !== "string"
         || val.match(/[=\r\n]/)
         || val.match(/^\[/)
         || (val.length > 1
             && isQuoted(val))
         || val !== val.trim() )
         ? JSON.stringify(val)
         : val.replace(/;/g, '\\;').replace(/#/g, "\\#")
}
```
- example usage
```shell
...
Alias for 'encode(object, [options])'

### safe(val)

Escapes the string 'val' such that it is safe to be used as a key or
value in an ini-file. Basically escapes quotes. For example

    ini.safe('"unsafe string"')

would result in

    "\"unsafe string\""

### unsafe(val)
...
```

#### <a name="apidoc.element.ini.stringify"></a>[function <span class="apidocSignatureSpan">ini.</span>stringify (obj, opt)](#apidoc.element.ini.stringify)
- description and source-code
```javascript
function encode(obj, opt) {
  var children = []
    , out = ""

  if (typeof opt === "string") {
    opt = {
      section: opt,
      whitespace: false
    }
  } else {
    opt = opt || {}
    opt.whitespace = opt.whitespace === true
  }

  var separator = opt.whitespace ? " = " : "="

  Object.keys(obj).forEach(function (k, _, __) {
    var val = obj[k]
    if (val && Array.isArray(val)) {
        val.forEach(function(item) {
            out += safe(k + "[]") + separator + safe(item) + "\n"
        })
    }
    else if (val && typeof val === "object") {
      children.push(k)
    } else {
      out += safe(k) + separator + safe(val) + eol
    }
  })

  if (opt.section && out.length) {
    out = "[" + safe(opt.section) + "]" + eol + out
  }

  children.forEach(function (k, _, __) {
    var nk = dotSplit(k).join('\\.')
    var section = (opt.section ? opt.section + "." : "") + nk
    var child = encode(obj[k], {
      section: section,
      whitespace: opt.whitespace
    })
    if (out.length && child.length) {
      out += eol
    }
    out += child
  })

  return out
}
```
- example usage
```shell
...

config.scope = 'local'
config.database.database = 'use_another_database'
config.paths.default.tmpdir = '/tmp'
delete config.paths.default.datadir
config.paths.default.array.push('fourth value')

fs.writeFileSync('./config_modified.ini', ini.stringify(config, { section: 'section' }))

This will result in a file called 'config_modified.ini' being written
to the filesystem with the following content:

[section]
scope=local
[section.database]
...
```

#### <a name="apidoc.element.ini.unsafe"></a>[function <span class="apidocSignatureSpan">ini.</span>unsafe (val, doUnesc)](#apidoc.element.ini.unsafe)
- description and source-code
```javascript
function unsafe(val, doUnesc) {
  val = (val || "").trim()
  if (isQuoted(val)) {
    // remove the single quotes before calling JSON.parse
    if (val.charAt(0) === "'") {
      val = val.substr(1, val.length - 2);
    }
    try { val = JSON.parse(val) } catch (_) {}
  } else {
    // walk the val to find the first not-escaped ; character
    var esc = false
    var unesc = "";
    for (var i = 0, l = val.length; i < l; i++) {
      var c = val.charAt(i)
      if (esc) {
        if ("\\;#".indexOf(c) !== -1)
          unesc += c
        else
          unesc += "\\" + c
        esc = false
      } else if (";#".indexOf(c) !== -1) {
        break
      } else if (c === "\\") {
        esc = true
      } else {
        unesc += c
      }
    }
    if (esc)
      unesc += "\\"
    return unesc
  }
  return val
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
