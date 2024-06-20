# Video/image converter

## About

Video and Image Converter is a web application built with Svelte and TypeScript that allows users to convert videos to MP4 format and image sequences to GIFs. The project leverages FFmpeg.wasm for performing the conversions directly in the browser without the need for server-side processing.

## Features

- Convert WebM/Mkv videos to MP4 format.
- Convert sequences of PNG images to GIFs.
- Drag and drop file support.
- Progress indicator for conversions.
- Beatiful confetti animation upon successful conversion.

## Installation

Clone the repository:

```bash
git clone https://github.com/SmolPeaCat/Video-Converter
cd Video-Converter
```

Install dependencies:

```bash
yarn install
```

Start the development server:

```bash
yarn run dev
```

Open the application in your browser:

```bash
Navigate to http://localhost:5000 (or the specified port).
```

## Usage

1. Drag and Drop Files:

   - Drag a WebM video or a sequence of PNG images into the drop area.

2. Conversion:

   - The application will automatically start the conversion process.
   - A progress bar will indicate the conversion progress.
   - Once the conversion is done, the file will be automatically downloaded (I have to change this).

### Example

[showcase](./src/assets/showcase.mkv)

## Technologies Used

- Svelte
- TypeScript
- FFmpeg.wasm: WebAssembly port of FFmpeg for performing media conversions.
- Neoconfetti: Confetti animation library.

## License

This project is licensed under the GPL V3.0 License. See the [LICENSE](./license.txt) file for details.
