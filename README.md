# NextNonce Documentation

This repository contains the source files for the official NextNonce documentation, which is built using [MkDocs](https://www.mkdocs.org/) with the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

## 📖 Live Documentation Site

The official, rendered documentation is available for everyone to view at:

**[https://docs.nextnonce.com](https://docs.nextnonce.com)**

This site includes:

  * **User Guide:** Step-by-step instructions on how to install and use the NextNonce mobile app.
  * **Developer Docs:** Information for developers who want to understand the project's architecture or contribute to the codebase.


## 🛠️ Building the Docs Locally

If you wish to contribute to the documentation or simply want to run a local instance of the site, you can do so by following these steps.

### Prerequisites

  * Python 3.x
  * `pip` (Python package installer)

### 1\. Clone the Repository

If you haven't already, clone this repository to your local machine:

```bash
git clone https://github.com/NextNonce/Docs.git
cd Docs
```

### 2\. Install Dependencies

The required Python packages are listed in the `requirements.txt` file. Install them using `pip`:

```bash
pip install -r requirements.txt
```

This will install MkDocs and all the necessary extensions and plugins.

### 3\. Run the Local Server

Once the dependencies are installed, you can start the live-reloading local server provided by MkDocs:

```bash
mkdocs serve
```

This command will build the site and start a local web server. By default, you can view the documentation by opening your web browser and navigating to:

**[http://127.0.0.1:8000](http://127.0.0.1:8000)**

Any changes you make to the Markdown files in the `docs/` directory will automatically trigger a rebuild, and the page in your browser will reload to show the updated content.