# File Preview Component for Vue

A professional Vue-based file preview solution supporting real-time rendering of multiple document formats.

![Vue](https://img.shields.io/badge/Vue-3.x-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)

## Overview

This Vue component provides seamless file preview capabilities for modern web applications, supporting popular formats including:

- Presentations (PPT/PPTX)
- PDF Documents
- Image files (PNG, JPG, GIF)
- Text files

Built upon [vue-file-viewer](https://github.com/zyl-ui/vue-file-viewer) with enhanced functionality and improved integration.

## Key Features

- **Multi-format Support**: Preview common office documents and images directly in browser
- **Hybrid Source Handling**: Supports both online resources and local BLOB data
- **Responsive Design**: Adaptive viewing experience across devices
- **Zero Dependencies**: Pure Vue implementation without external library requirements

## Installation

### Prerequisites

- Vue 3.x project
- Static resource server configuration

### Implementation Steps

1. **Resource Deployment**  
   Copy the `file-viewer` directory to your project's `public` folder.

2. **Component Registration**  
   Import the preview component in your Vue module:
   ```javascript
   import PreviewFile from "./components/PreviewFile.vue";
   ```
