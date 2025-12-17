# Glossary

This document provides definitions for domain-specific terminology and obscure variable names found in the Storm-Breaker codebase.

## Project Overview

**Storm-Breaker** is a social engineering tool designed to obtain device information, location, webcam images, and microphone recordings from a target user. It typically involves running a local PHP server, exposing it via Ngrok, and sending a phishing link to the target.

## General Terms

*   **Ngrok**: A cross-platform application that enables developers to expose a local development server to the Internet with minimal effort. Storm-Breaker uses it to make the local phishing pages accessible to external targets.
*   **Listener**: Refers to the background process (specifically the PHP built-in server started by `st.py`) that waits for and handles incoming connections and data submissions from victims.
*   **Web Panel**: The web-based graphical user interface (GUI) hosted locally (usually on port 2525), allowing the attacker to manage the tool, view captured logs, and control the listener.
*   **Templates**: Pre-designed HTML/PHP pages that mimic legitimate services (e.g., Weather, Near You) to trick users into granting permissions (camera, microphone, location).

## Python Components (`st.py` & `modules/`)

*   **`st.py`**: The main entry point script for the application. It handles dependency checks, updates, and starts the PHP server.
*   **`modules/control.py`**: Contains functions to control the PHP server process, such as `run_php_server` and `kill_php_proc`.
*   **`modules/banner.py`**: Responsible for displaying the ASCII art banner when the tool starts.
*   **`modules/check.py`**: Handles dependency verification and checking for updates.
*   **`md5_hash`**: A function in `control.py` (and the resulting hash string) used to generate unique filenames for PHP server logs based on the current timestamp.
*   **`OsName`**: A variable in `banner.py` that stores the operating system name (e.g., "Windows", "Linux"), used to determine the correct command to clear the console.

## Web Panel & PHP Structure (`storm-web/`)

*   **`storm-web/`**: The root directory for the web interface and phishing resources.
*   **`Settings.json`**: A JSON file used to persist state information, specifically the Process ID (`pid`) of the PHP server, the start status (`is_start`), and the current version (`version`). Note: There is also a `Settings.json` in the root directory, which appears to be a duplicate or template.
*   **`check-c.json`**: A JSON file containing authentication details for the web panel, specifically a `token` and an `expired` status. The name likely stands for "Check Credentials" or "Check Config".
*   **`config.php`**: Stores the administrative credentials (`admin` username and password) for accessing the Web Panel.
*   **`receiver.php`**: A PHP script that aggregates captured data from the `result.txt` files located in each template directory and sends it to the Web Panel's frontend.
*   **`list_templates.php`**: A script that scans the `templates/` directory and returns a JSON list of available phishing templates.
*   **`logindata`**: The name of the HTTP cookie used to store the session token for the Web Panel authentication.
*   **`IAm-logined`**: A PHP session variable used to track if the user is currently logged in to the Web Panel.
*   **`ClientJS`**: A JavaScript library (`client.min.js`) included in templates to fingerprint the victim's browser, OS, and device information.
*   **`WarpSpeed`**: A JavaScript library (`warpspeed.min.js`) used in the `nearyou` template to create a visual starfield background effect.

## Templates & Data Capture Variables

*   **`result.txt`**: A text file present within each template directory (e.g., `storm-web/templates/camera_temp/result.txt`). It acts as a temporary buffer where captured data (location, image paths, etc.) is written by `handler.php` or `post.php` before being read by `receiver.php`.
*   **`handler.php`**: A PHP script found in most templates. It receives data via POST requests (usually location or device info) and writes it to `result.txt`.
*   **`post.php`**: A PHP script found in the `camera_temp` template. It specifically handles the reception of base64-encoded image data, decodes it, saves it as a PNG file, and logs the path to `result.txt`.
*   **`upload.php`**: A PHP script found in the `microphone` template. It handles the upload of audio files (WAV format).
*   **`cat`**: An obscure POST parameter name used in `camera_temp/index.html` (Javascript) and `camera_temp/post.php` to transmit the base64-encoded image data.
*   **`imgdata`**: A JavaScript variable in `camera_temp/index.html` holding the base64 string of the captured webcam image.
*   **`audio_data`**: The key used in `microphone/js/_app.js` and `microphone/upload.php` to identify the uploaded audio file in the `FormData`.
*   **`check_brave`**: A variable in `storm-web/assets/js/loc.js` used to detect if the victim is using the Brave browser, which often blocks fingerprinting attempts.
