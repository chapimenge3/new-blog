---
title: "Golang Wasm on Cloudflare Workers"
meta_title: "Golang Wasm on Cloudflare Workers"
date: 2024-05-19 10:23:39
image: "/images/blogs/programming/goasm-cf-worker.png"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["programming", "golang", "wasm", "cloudflare"]
draft: true
---

Golang WebAssembly (Wasm) on Cloudflare Workers is a powerful combination that allows you to run your Go code on the edge. This article will show you how to compile a simple Go program to Wasm and deploy it to Cloudflare Workers.

{{< toc >}}

Hey there! In this article, we will explore how to run Golang WebAssembly (Wasm) on Cloudflare Workers. This is a powerful combination that allows you to run your Go code on the edge. We will compile a simple Go program to Wasm and deploy it to Cloudflare Workers.

# Introduction

Cloudflare Workers is a serverless platform that allows you to run JavaScript, Typescript, Rust, and Python code on the edge. With the addition of WebAssembly support, you can now run Wasm modules on Cloudflare Workers using your favorite programming language. This opens up a whole new world of possibilities, as you can now run code written in any language that compiles to Wasm on Cloudflare Workers.

In this article, we will build a very simple Go Program that will convert PNG to/from JPEG using the `image` package from the Go standard library. We will then compile this program to Wasm and deploy it to Cloudflare Workers.

# Prerequisites

This articles assumes you have hands-on experience with Go programming language and Cloudflare Workers. If you are new to Go, you can check out the [official Go documentation](https://golang.org/doc/). If you are new to Cloudflare Workers, you can check out the [official Cloudflare Workers documentation](https://developers.cloudflare.com/workers/).

# Setting up the Go Project

Let's start by setting up a new Vite Project following with simple Golang code.
    
```bash
npm create vite@latest go-wasm-cloudflare  -- --template vanilla
cd go-wasm-cloudflare
npm install
npm run dev
```

The above will create a new vite project and start the development server. You can also use your own vanila HTML, CSS, and JS setup. I just prefer using Vite for faster development.

Now let's do a very simple UI to upload an image and convert it to PNG or JPEG.

```html
<!doctype html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <link rel="icon" type="image/svg+xml" href="/logo.png" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@1.0.0/css/bulma.min.css">
  <title>Image Tool - GOLANG webassembly POC</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css"
    integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLJAJ9/2PkPKZ5QiAj6Ta86w+fsb2TkcmfRyVX3pBnMFcV7oQPJkl9QevSCWr3W6A=="
    crossorigin="anonymous" referrerpolicy="no-referrer" />
  <link href="https://fonts.googleapis.com/css?family=Open+Sans" rel="stylesheet">
  <style type="text/css">
    html,
    body {
      font-family: 'Open Sans';
    }
  </style>
</head>

<body>
  <div id="app">
    <section class="hero is-fullheight is-default is-bold">
      <div class="hero-body">
        <div class="container">
          <div class="columns is-vcentered">
            <div class="column is-6 is-offset-1">

              <div class="select is-normal mb-2">
                <select id="to_format">
                  <option>Convert to</option>
                  <option value="png">PNG</option>
                  <option value="jpeg">JPEG</option>
                </select>
              </div>

              <div id="file-js-example" class="file has-name">
                <label class="file-label">
                  <input class="file-input" type="file" id="image" name="image" accept="image/*">
                  <span class="file-cta">
                    <span class="file-icon">
                      <i class="fas fa-upload"></i>
                    </span>
                    <span class="file-label"> Choose a Image </span>
                  </span>
                  <span class="file-name"> No Image uploaded </span>
                </label>
              </div>

              <br>
              <p class="has-text-centered">
                <a class="button is-medium is-primary is-outlined" id="convert">
                  Convert
                </a>
              </p>
            </div>


            <div class="column is-5">
              <figure class="image is-fullwidth">
                <img id="final-out" src="https://dummyimage.com/600x400/000/fff?text=PLACEHOLDER IMG" alt="Description">
              </figure>
              <progress id="loading" style="visibility: hidden;" class="progress is-small is-primary" max="100">15%</progress>
              <p class="has-text-centered">
                <button class="button is-medium is-primary is-muted" id="download-img">
                  Downlaoad
                </button>
              </p>
            </div>

          </div>
        </div>
      </div>
    </section>
  </div>
</body>

</html>
```

What the above code does is a very simple UI with the below screenshot(Dark reader might be applied).

{{< image src="images/blogs/programming/go-wasm-1-sample.png" caption="" alt="Simple File Upload UI " height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="File upload UI"  webp="false" zoomable="true" >}}

With that i think we don't need any more fancy UI. Let's move on to the Go code.

First we need to create a module for the Go code.

```bash
go mod init go-wasm-cloudflare
touch main.go
```

Now let's write the Go code.

```go
package main

import (
	"bytes"
	"fmt"
	"image/jpeg"
	"image/png"
	"io"
	"syscall/js"
)

// convertToPNG converts a JPEG image to PNG
// and writes the result to the given writer.
// Returns an error if the conversion fails.
func convertToPNG(w io.Writer, r io.Reader) error {
	img, err := jpeg.Decode(r)
	if err != nil {
		return err
	}
	return png.Encode(w, img)
}

// convertToJPEG converts a PNG image to JPEG
// and writes the result to the given writer.
// Returns an error if the conversion fails.
func convertToJPEG(w io.Writer, r io.Reader) error {
	img, err := png.Decode(r)
	if err != nil {
		return err
	}
	return jpeg.Encode(w, img, &jpeg.Options{Quality: 100})
}

// convertJPEGToPNG is a JavaScript function that converts a JPEG image to PNG.
// It takes a Uint8Array containing the JPEG image data as input and returns a Uint8Array containing the PNG image data.
// The function signature is: convertJPEGToPNG(input: Uint8Array, toType: string): Uint8Array
func convertJPEGToPNG(this js.Value, args []js.Value) interface{} {
	fmt.Println("Go: File Recieved")
	input := make([]byte, args[0].Get("length").Int()) // Get the length of the input array
	// we copy the input array to a Go byte slice so that we can work with it more easily
    js.CopyBytesToGo(input, args[0])
	toType := "png"
	if len(args) > 1 {
		toType = args[1].String()
	}

	fmt.Println("Go: toType", toType)
	
    var out_img bytes.Buffer // Create a buffer to store the output image
	if toType == "png" {
		convertToPNG(&out_img, bytes.NewReader(input)) // see here we pass pointer to the buffer to write the output
	} else if toType == "jpeg" {
		convertToJPEG(&out_img, bytes.NewReader(input))
	}
    // by this time the out_img should have the converted image data
    // so we will copy the data to a Uint8Array and return it
	js_out := js.Global().Get("Uint8Array").New(len(out_img.Bytes()))
	// now we will copy the data from the go buffer to the js Uint8Array
    js.CopyBytesToJS(js_out, out_img.Bytes())
	return js_out
}

func main() {
    // We will create an empty channel to keep the program running
    // because the main function will exit immediately after the program starts
    // to keep the program running we will block the main function using the channel
    // so that the program does not exit
	c := make(chan struct{}, 0)
	js.Global().Set("convertJPEGToPNG", js.FuncOf(convertJPEGToPNG))
	<-c
}
```

The above code is a simple Go code that converts a JPEG image to PNG and vice versa. The `convertJPEGToPNG` function is a JavaScript function that takes a `Uint8Array` containing the JPEG image data as input and returns a `Uint8Array` containing the PNG image data. The function signature is `convertJPEGToPNG(input: Uint8Array, toType: string): Uint8Array`.

The `main` function sets the `convertJPEGToPNG` function as a global function in the JavaScript environment. It then blocks the main function using a channel so that the program does not exit immediately after starting.

Now let's compile the Go code to Wasm.

```bash
GOOS=js GOARCH=wasm go build -o main.wasm
```

The above command compiles the Go code to Wasm and outputs the Wasm binary to `main.wasm`.

Before we move on to the js code that will interact with the Go code, we need to copy a `wasm_exec.js` file from the Go installation directory to the public directory of our project.

```bash
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" ./public
```

This javascript file is required to run the Wasm binary in the browser.

Now let's write the JavaScript code that will interact with the Go code.

Create a new file `main.js` and add the following code.

```javascript

/*
 * This function converts a JPEG image to PNG or vice versa.
*/
async function convert() {
  // we calculate how long it takes just to see the performance
  const startTime = new Date().getTime();
  const input = document.getElementById('image');
  const to_format = document.getElementById('to_format').selectedOptions[0].value

  if (input.files.length === 0 || !to_format || to_format === 'Convert to') {
    alert('Make sure to select a file and format')
    return
  }

  const img = input.files[0];
  if (img.type === `image/${to_format}`) {
    alert('The image is already in the selected format')
    return
  }
  document.getElementById("loading").style.visibility = "visible";

  // Now we have the image file, we will read the file as an array buffer
  // so that we can pass it to the Go function
  const buff = await img.arrayBuffer();
  const data = new Uint8Array(buff);
  const png = convertJPEGToPNG(data, to_format) // This is the Go function that we have defined in the main.go file
  const endTime = new Date().getTime();
  const total = endTime - startTime;
  // now the go function have returned the converted image data and completed its work
  
  const blob = new Blob([png], {
    type: 'image/png'
  })
  const url = URL.createObjectURL(blob)
  document.getElementById('final-out').src = url
  document.getElementById("loading").style.visibility = "hidden";
  document.getElementById('download-img').removeAttribute('disabled')

}

/*
 * This function is helper to downloads the current image displayed on the screen
*/
function downloadCurrentImage() {
  console.log('Downoading')
  const img = document.getElementById('final-out')
  // check if the img is available or it is not starts with 'blob:'
  if (!img || !img.src || !img.src.startsWith('blob:')) {
    alert('No image available')
    return
  }
  const a = document.createElement('a')
  const url = img.src
  const filename = 'image.png'
  a.href = url
  a.download = filename
  a.click()
}



// listen for file upload and update the file name
const fileInput = document.querySelector("#file-js-example input[type=file]");
fileInput.onchange = () => {
  if (fileInput.files.length > 0) {
    const fileName = document.querySelector("#file-js-example .file-name");
    fileName.textContent = fileInput.files[0].name;
  }
};

// listen for convert button click to start the conversion
document.getElementById("convert").addEventListener("click", convert);
document.getElementById("download-img").addEventListener("click", downloadCurrentImage)
```

Am sure the above code isn't that hard to understand except the byte conversion part incase you are not familiar with bytes and buffers. The code is well commented and should be easy to understand.

Now let's update the `index.html` to serve the wasm file and the javascript file.

```html
<!doctype html>
<html lang="en">
<head>
    ...
    <script src="./wasm_exec.js"></script>
    <script>
        if (WebAssembly) {
        const go = new Go();
        WebAssembly.instantiateStreaming(fetch('/main.wasm'), go.importObject).then((result) => {
            go.run(result.instance)
            console.log("Wasm loaded")
        })

        }
    </script>
    ...
</head>
<body>
    ...
    <script src="/main.js"></script>
</body>
```

The above code loads the `wasm_exec.js` file and then loads the `main.wasm` file using the `WebAssembly.instantiateStreaming` function. It then runs the Wasm binary using the `go.run` function in a near native speed. The `main.js` file is then loaded to interact with the Go code.

Now if everything is setup correctly, you should be able to run the project and see the UI. You can now upload an image and convert it to PNG or JPEG.

We are about to deploy the project to cloudflare workers which is going to be extremely easy because we don't have to do much. 

Commit the changes to git and push to your repository.

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

{{< accordion "Are you new to git?" >}}

- Go to github.com and create a new repository. 
- If you setup SSH key in your local copy the 

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-repo-url>
```
{{< /accordion >}}