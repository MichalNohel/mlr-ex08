# Leaf lens — MLR exercise 08

A buildless leaf classifier: one `index.html` with plain JavaScript, ONNX Runtime Web 1.20.1 from jsDelivr, and the supplied ONNX model and metadata. All inference runs on the visitor’s device.

**Public page URL:** https://michalnohel.github.io/mlr-ex08/

The page displays this app URL beside a locally generated QR code.

## Use

Open the HTTPS page in Safari on an iPhone, tap **Start rear camera**, grant camera permission, and center a leaf. All four class probabilities update continuously. **Stop camera** releases the camera. **Choose a photo** works without camera permission. The preview uses the same central square as inference.

For local desktop use, serve this folder with `python -m http.server 8000` and open `http://localhost:8000/`. Opening the HTML directly as a file will not work because model loading uses fetch. A phone needs an HTTPS deployment for camera access.

## Model pipeline

Read `model_info.json` for class order, input size, and channel mean/std. Crop the central square of each source image, resize to 96×96 using the browser’s canvas, then convert RGB bytes to float values with `(pixel / 255 - mean[channel]) / std[channel]`. Store channels in a float32 NCHW tensor `[1, 3, 96, 96]`, supplied to the model’s `image` input. Apply numerically stable softmax to its four output logits. The page uses single-threaded WebAssembly, `playsinline` video, and user-triggered camera access for Safari compatibility.

## Verification

Click **Check reference image** to run `check_image.jpg` through the app’s actual pipeline and compare every probability against `check_expected.json`. The page reports actual and expected probabilities, maximum absolute error, and PASS/FAIL using absolute tolerance 0.0001.

Chromium and Playwright WebKit both returned:

| Class | Browser probability | Expected probability |
| --- | ---: | ---: |
| apple | 0.0444096161 | 0.0444102362 |
| cherry | 0.5269498626 | 0.5269479752 |
| chestnut | 0.0063746888 | 0.0063746599 |
| maple | 0.4222658325 | 0.4222670496 |

Maximum absolute error: 0.0000018875. Native ONNX Runtime independently matched within 0.00000012. Photo upload passed in Chromium and WebKit; live inference and stop passed with Chromium’s simulated camera. A physical iPhone camera has not been tested.

## QR code

The footer bundles the MIT-licensed QRCode generator by Kazuhiko Arase (`qrcode-generator` 1.4.4). It generates the QR image entirely inside the browser. Its payload is the current app URL with query/fragment removed and `index.html` normalized to its containing directory. The adjacent clickable link uses the identical URL. No external QR-image service is used.

## Hosting

Publish this folder in the public repository `mlr-ex08`, then enable GitHub Pages from `main` at `/` (root). No build tools or server-side application are required. Include `leaf_model.onnx`, `model_info.json`, `check_image.jpg`, and `check_expected.json` beside `index.html`.

ONNX Runtime documentation: [browser support](https://onnxruntime.ai/docs/get-started/with-javascript/web.html), [WebAssembly configuration](https://onnxruntime.ai/docs/tutorials/web/env-flags-and-session-options.html).
