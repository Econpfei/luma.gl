# Writing Modern WebGL

A major design goal of luma.gl is to make it easy to write portable applications that work with the latest WebGL APIs (WebGL2 components and methods in JavaScript and GLSL 3.00 ES in shaders), by providing a modern JavaScript components and transparently handling differences between WebGL and GLSL versions.


## WebGL Version Incompatibilities

While WebGL2 claims to be almost 100% backwards compatible with WebGL1, this is really only true for the core JavaScript side of the API. There are two reasons for this:

* WebGL Extensions - To use WebGL1 extensions to match functionality provided by default in WebGL2, one is forced to work with two different APIs, since WebGL2 explicitly forbids implementation of extensions covered by the WebGL2 API.

* GLSL versions - And if an application wants to or needs to a specific GLSL version (whether GLSL 1.00 to remain WebGL1 compatible, or GLSL 3.00 to use some new feature or just write in a modern style), all shader code (including imported shader modules) must be updated since basic parts of the syntax have changed.

If an application only needs to support WebGL2 browsers (and does not need to run under e.g. headless gl etc), or is satisfied with the WebGL1 subset of the API (and GLSL 1.00 shader syntax), then no support is needed. But if not, luma.gl provides solutions in the form of capability detection, API polyfills and automatic GLSL code transpilation.


## Capability Detection and API Polyfills

TBA


Partial table of WebGL2 Features that can be used in WebGL1 using the common WebGL2 API.

| VertexArrays        | Always available |
| Instanced Rendering |                  |
| ...                 |                  |


## Automatic GLSL Code Transpilation

If one wishes to use GLSL 3.00 features in a shader, one has to make modifications to the shader so that it no longer runs on GLSL 1.00 code. For a small monolithic shader it may be less of an issue (e.g. if you need WebGL2 features, use GLSL 3.00, otherwise GLSL 1.00) but if the goal is to create reusable shader code that can run under both WebGL1 and WebGL2 and both in GLSL 1.00 and GLSL 3.00 shaders, then there are problems.


### Textual replacement

The required modifications are very simple, and can mostly be done by a (fairly) simple series of textual replacements, as detailed in the proposal below.


### Fragment Shader Outputs

The major complication is related to fragment shader outputs, that use built-in `gl_fragColor`/`gl_fragData[]` variables in GLSL 1.00, but must be declared as variables in GLSL 3.00. And any variable starting with `gl_` is reserved, so names must be changed.


* Shader Assembler knows the target version of GLSL.
* It applies as series of regular expressions (different between vertex and fragment shader) to replace basic constructs.
* Users will need to avoid WebGL2 constructs if they wish their shaders to compile


### Table of replacements

#### Converting from GLSL 3.00 to 3.00

To enable portable code to be written, we must allow some non-conformant code in GLSL 3.00.

Most importantly, `texture` in GLSL 3.00 might translate to either `texture2D` or `textureCube` in GLSL 1.00. Therefor we allow `textureCube` to be used in GLSL 3.00 code but automatically change it to `texture`.


#### Converting from GLSL 3.00 to 1.00

Vertex Shader

* `in` -> `attribute` - only beginning of line
* `out` -> `varying` - only beginning of line

Fragment Shader

* `in` -> `varying` - only beginning of line
* `out` -> `gl_fragColor/gl_fragData[]` - only beginning of line, remove declaration

Common

* `texture` -> `texture2D`


### Converting from GLSL 1.00 to 3.00

Vertex Shader

* `attribute` -> `in` - can be done with macro
* `varying` -> `out` - can be done with macro

Fragment Shader

* `varying` -> `out` - only beginning of line
* `out` -> `gl_fragColor/gl_fragData[]` - only beginning of line, remove declaration, if multiple `out`s use gl_fragData

Common

* `texture2D` -> `texture`
* `textureCube` -> `texture`
