# System Architecture

## Overview

"Storm Breaker" is a social engineering tool designed to obtain device information, location, webcam images, and microphone recordings from a victim's device. It operates by hosting a local web server (PHP) and using a Python script to manage the process and provide a command-line interface (or simple runner) for the user.

The system is a hybrid application combining a **Python backend** (process management, dependencies check) and a **PHP frontend/backend** (serving attack templates, receiving captured data, and an admin panel).

## High-Level Design

The architecture consists of three main components:

1.  **Python Runner (`st.py` & `modules/`)**:
    *   Acts as the entry point.
    *   Checks for dependencies (PHP, Python libraries).
    *   Manages the lifecycle of the PHP web server (starts/stops).
    *   Updates configuration (`Settings.json`) with process IDs.

2.  **PHP Web Application (`storm-web/`)**:
    *   **Admin Panel**: Provides a web interface for the attacker to view captured data (logs, images, audio).
    *   **Templates**: Social engineering pages (e.g., fake camera apps, microphone recorders) served to the victim.
    *   **Data Receivers**: PHP scripts (`post.php`, `upload.php`) that receive data posted from the victim's browser.

3.  **Client-Side Scripts (JavaScript)**:
    *   **Victim-side**: Scripts embedded in templates (`recorder.js`, `loc.js`) to capture media and location via browser APIs (`getUserMedia`, `Geolocation`).
    *   **Admin-side**: Scripts (`script.js`) to poll the server for new data and update the UI.

## Data Flow

1.  **Initialization**:
    *   User runs `st.py`.
    *   `st.py` starts a PHP server on `localhost:2525`.
    *   The PID of the PHP process is stored in `storm-web/Settings.json`.

2.  **Attack Phase**:
    *   Attacker exposes the local server (via Ngrok or manual hosting) and sends a link to the victim.
    *   Victim visits a template page (e.g., `/templates/camera_temp/index.html`).
    *   **Data Capture**:
        *   **Images**: JavaScript captures video frames and POSTs base64 data to `post.php`.
        *   **Audio**: JavaScript records audio and uploads `.wav` files to `upload.php`.
        *   **Location**: JavaScript captures coordinates and sends them (likely via AJAX) to the server.
    *   **Storage**: PHP scripts save files to `storm-web/images/` or `storm-web/sounds/` and append a notification message to `result.txt` within the specific template directory.

3.  **Monitoring Phase**:
    *   Attacker views the Admin Panel (`panel.php`).
    *   `script.js` polls `receiver.php` every few seconds.
    *   `receiver.php` scans template directories for `result.txt`.
    *   If data exists, it returns the content to `script.js` and clears the `result.txt` file.
    *   The Admin Panel displays a notification and adds the entry to the log textarea.

## Key External Dependencies

*   **Python 3**: For the runner script.
*   **PHP**: For the web server and backend logic.
*   **Ngrok** (Optional but recommended): For exposing the local server to the internet.
*   **Python Libraries**:
    *   `requests`: For checking updates.
    *   `colorama`: For colored CLI output.
    *   `psutil`: For process management (though `subprocess` is also heavily used).
*   **JavaScript Libraries**:
    *   `jQuery`: Used extensively in both admin panel and templates.
    *   `SweetAlert2` / `GrowlNotification`: For UI notifications in the admin panel.

## Non-Standard Patterns

### 1. Python as a PHP Process Manager
The application uses Python primarily to spawn and kill a PHP process (`php -S localhost:2525`). This is an unusual pattern where Python wraps a web application that could theoretically run standalone.

### 2. Built-in Development Server for Production
The tool relies on PHP's built-in development server (`php -S`), which is single-threaded and not intended for production use. This fits the "single-user tool" nature of the project but is non-standard for robust web applications.

### 3. File-Based IPC (Inter-Process Communication)
*   **PHP <-> Admin Panel**: Communication happens via the filesystem. Victim-facing scripts write to `result.txt`, and the admin-facing script (`receiver.php`) reads and clears this file. There is no database.
*   **Python <-> PHP**: State (PID, start status) is shared via `storm-web/Settings.json`.

### 4. Direct Process Killing
The Python script manages the PHP server by storing its PID in a JSON file and issuing `kill -9` commands to terminate it. This is a brittle form of process management compared to standard signaling or service managers.

### 5. Mixed Codebase (Polyglot)
The logic is split between Python (CLI/Setup) and PHP (Application), with no direct communication other than the filesystem.
