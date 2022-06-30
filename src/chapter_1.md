# Two and Two is ... Wasm?

main.c
```c
int add(a, b) {
  return a + b;
}
```

Let's try to get this to compile and run ...  in our web browser!

First, we'll need to compile it to a `.wasm` file.

You can use clang if you want, there's great resources for that [here](https://surma.dev/things/c-to-webassembly/).

However, I found that the commands that are supplied there ... don't consistently work across platforms.

Plus, clang is clunky. Let's use [zig](https://ziglang.org) to wrap up that whole LLVM mess for us, into one reasonable, consistent executable.

```
vim main.c

mkdir build
cd build

zig build-lib \
  --export add \
  -dynamic -target wasm32-freestanding ../main.c
```

## Huh?

Here's a breakdown of all of the parts of that command:
- `build-lib`: Wasm files are just a collection of exported memory and functions, so they're libraries!
- `--export add`: that's the function we want to be able to call from the web browser! in the next chapter, I'll show how to have zig do this for us so we don't have to list each one we want to export :)
- `-dynamic`: static libraries are known at compile time. Wasm libraries are dynamic, because you can't reach across library boundaries until runtime. This gives them lots of fun properties, like hot-swappability!
- `-target wasm32-freestanding`: good ole 32 bit web assembly, without any kind of operating system helpers, so "freestanding." Keepin' things lean and mean >:)


## Cute Lil Guy :>
```
test % wc -c build/main.wasm
      85 build/main.wasm
```
As you can see, our Wasm file is only 85 bytes in size :O

## One last file ...
Great! Now let's try to use that Wasm file in our web browser.

Fortunately, there's a [handy function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiateStreaming) for that :)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Useful C Wasm without Emscripten</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <meta name="description" content="" />
  </head>
  <body>
    <script>
      (async () => {
        const wasm = fetch("build/main.wasm");
        const { instance } =
          await WebAssembly.instantiateStreaming(wasm, { env: {} });

        console.log(instance.exports.add(2, 2));
      })()
    </script>
  </body>
</html>
```

Now your file system should look like this:
```
test/
  build/
    main.wasm
  main.c
  main.html
```

## MIME woes

Unfortunately, you can't just open that HTML file in your web browser. It needs to grab a file from your file system -- `build/main.wasm` --  and Chrome won't let it do that unless it's hosted from a web server.

And, the static file servers you might typically use, like `npx live-server` or `python3 -m http.server` ... well, they actually treat Wasm files like random binaries, instead of marking them with the `application/wasm` MIME type, which means ... you can't load them :(

Fortunately, there are still a couple static file servers I know of that correctly mark Wasm files. Try `npx wasm-server` and then go to `localhost:3000/main.html`

# Hello, 4!
And lo and behold, if you look in your console ... the number 4 has been printed!
