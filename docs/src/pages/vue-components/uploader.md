---
title: Uploader
---
Quasar supplies a way for you to upload files through the QUploader component.

## Installation
<doc-installation components="QUploader" />

## Usage

::: warning
QUploader requires a back-end server to receive the files. The examples below will not actually upload.
:::

::: tip
QUploader is `drag and drop` compliant.
:::

::: warning
When using vee-validate, you have to rename the "fieldBagName" configuration of vee-validate for the q-uploader to work.
:::

### Design

<doc-example title="Basic" file="QUploader/Basic" />

<doc-example title="Dark" file="QUploader/Dark" />

### Uploading multiple files

By default, multiple files will be uploaded individually (one thread per file). Should you want all files to be uploaded in a single thread, use the `batch` property (second QUploader in the example below).

<doc-example title="Multiple" file="QUploader/Multiple" />

### Restricting upload

<doc-example title="Basic restrictions" file="QUploader/RestrictionBasic" />

::: tip
In the example above, we're using `accept` property. Its value must be a comma separated list of unique file type specifiers. Maps to 'accept' attribute of native input type=file element. [More info](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#Unique_file_type_specifiers).
:::

::: warning
Recommended format for the `accept` property is `<mediatype>/<extension>`. Examples: "image/png", "image/png". QUploader uses an `<input type="file">` under the covers and it relies entirely on the host browser to trigger the file picker. If the `accept` property (that gets applied to the input) is not correct, no file picker will appear on screen or it will appear but it will accept all file types.
:::

You can also apply custom filters (which are executed after user picks files):

<doc-example title="Filter" file="QUploader/RestrictionFilter" />

### Adding headers

Use `headers` for setting additional XHR headers to be sent along the upload request. Also check `form-fields` prop in the API, if you need additional fields to be embedded.

<doc-example title="Headers" file="QUploader/Headers" />

::: tip
These two props (`headers` and `form-fields`) can be used as a function too (`(files) => Array`), allowing you to dynamically set them based on the files that are to be uploaded.
:::

There is also the `with-credentials` property, which sets `withCredentials` to `true` on the XHR used by the upload process.

### Handling upload

<doc-example title="Auto upload on file selection" file="QUploader/UploadAuto" />

<doc-example title="Custom upload URL" file="QUploader/UploadURL" />

::: tip
You can also customize the HTTP headers and HTTP method through `headers` and `method` props. Check QUploader API section.
:::

### Factory function
There is a `factory` prop you can use which must be a Function. This function can return either an Object or a Promise resolving with an Object (and in case the Promise fails, `@factory-failed` event is emitted).

The Object described above can override the following QUploader props: `url`, `method`, `headers`, `formFields`, `fieldName`, `withCredentials`, `sendRaw`). The props of this Object can be Functions as well (of form `(file[s]) => value`):

<doc-example title="Promise-based factory function" file="QUploader/FactoryPromise" />

You can also use the `factory` Function prop and return immediately the same Object. This is useful if you want to set multiple props (described above) simultaneously:

<doc-example title="Immediate return factory function" file="QUploader/FactoryImmediate" />

### Slots

In the example below we're showing the equivalent of the default header. Also notice some Boolean scope properties that you can use: `scope.canAddFiles`, `scope.canUpload`, `scope.isUploading`.

::: warning
Notice that you must install and use one more component (QUploaderAddTrigger) in order to be able to add files to the queue. This component needs to be placed under a DOM node which has `position: relative` (hint: QBtn has it already) and will automatically inject the necessary events when user clicks on its parent (do NOT manually add `@click="scope.pickFiles"`).
:::

::: tip IE11 Support with custom header
For the file picker to work on IE11 when wrapping QUploaderAddTrigger with a QBtn, make sure that this button has `type="a"` specified.
:::

<doc-example title="Custom header" file="QUploader/SlotHeader" />

<doc-example title="Custom files list" file="QUploader/SlotList" />

## Server endpoint

QUploader works by default with the HTTP(S) protocol to upload files (but it's not limited to it as you'll see in the section following this one).

### Nodejs

Below is a basic server example written in Nodejs. It does nothing other than receiving the files, so consider it as a starting point.

::: tip
It is by no means required to use a Nodejs server like above -- you can handle file upload however you want, as long as the method you are using fits the HTTP protocol. Example with [PHP](https://secure.php.net/manual/en/features.file-upload.php).
:::

```js
const
  express = require('express'),
  app = express(),
  formidable = require('formidable'),
  path = require('path'),
  fs = require('fs'),
  throttle = require('express-throttle-bandwidth')

const
  port = process.env.PORT || 4444,
  folder = path.join(__dirname, 'files')

if (!fs.existsSync(folder)) {
  fs.mkdirSync(folder)
}

app.set('port', port)
app.use(throttle(1024 * 128)) // throttling bandwidth

app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept')
  next()
})

app.post('/upload', (req, res) => {
  const form = new formidable.IncomingForm()

  form.uploadDir = folder
  form.parse(req, (_, fields, files) => {
    console.log('\n-----------')
    console.log('Fields', fields)
    console.log('Received:', Object.keys(files))
    console.log()
  })
  res.send('Thank you')
})

app.listen(port, () => {
  console.log('\nUpload server running on http://localhost:' + port)
})
```

### Spring

Below is a [Spring](https://spring.io/guides/gs/uploading-files/) example. Attribute `fieldName="file"` is mapping with `@RequestPart(value = "file")`.

```
// java
@RestController
public class UploadRest {
	@PostMapping("/upload")
	public void handleFileUpload(@RequestPart(value = "file") final MultipartFile uploadfile) throws IOException {
		saveUploadedFiles(uploadfile);
	}

	private String saveUploadedFiles(final MultipartFile file) throws IOException {
		final byte[] bytes = file.getBytes();
		final Path path = Paths.get("YOUR_ABSOLUTE_PATH" + file.getOriginalFilename());
		Files.write(path, bytes);
	}
}

// html
<q-uploader fieldName="file" url="YOUR_URL_BACK/upload" with-credentials />
```

## Supporting other services
QUploader currently supports uploading through the HTTP protocol. But you can extend the component to support other services as well. Like Firebase for example. Here's how you can do it.

Below is an example with the API that you need to supply. You'll be creating a new Vue component that extends the Base of QUploader that you can then import and use in your website/app.

::: tip
For the default XHR implementation, check out [source code](https://github.com/quasarframework/quasar/blob/dev/ui/src/components/uploader/uploader-xhr-mixin.js).
:::

::: warning Help appreciated
We'd be more than happy to accept PRs on supporting other upload services as well, so others can benefit.
:::

```js
// MyUploader.js
import { QUploaderBase } from 'quasar'

export default {
  name: 'MyUploader',

  mixins: [ UploaderBase ],

  computed: {
    // [REQUIRED]
    // we're working on uploading files
    isUploading () {
      // return <Boolean>
    },

    // [optional]
    // shows overlay on top of the
    // uploader signaling it's waiting
    // on something (blocks all controls)
    isBusy () {
      // return <Boolean>
    }
  },

  methods: {
    // [REQUIRED]
    // abort and clean up any process
    // that is in progress
    abort () {
      // ...
    },

    // [REQUIRED]
    upload () {
      if (this.canUpload === false) {
        return
      }

      // ...
    }
  }
}
```

For the UMD version, you can extend `Quasar.components.QUploaderBase`.

## QUploader API
<doc-api file="QUploader" />
