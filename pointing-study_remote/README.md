# Mouse vs. Smartphone Tilt Pointing Study (Web / Remote Version)

A web-based quantitative Human-Computer Interaction (HCI) experiment designed to evaluate and compare pointing efficiency between a traditional computer mouse and smartphone tilt interaction using **Fitts' Law (1954)**.

---

## 📌 Project Overview

Pointing is one of the most fundamental actions in graphical user interfaces. While desktop setups rely heavily on the mouse as an industry baseline, modern smartphones pack precise motion sensors (gyroscopes and accelerometers). 

This remote web application tests whether **smartphone tilt input** (used as a wireless remote) can serve as a viable or competitive pointing modality. By running a reciprocal target-acquisition task across varying distances ($A$) and target widths ($W$), the study logs performance metrics—including Movement Time ($MT$), Error Rate, and Throughput ($TP$)—to draw a rigorous comparison against standard mouse input.

---

## 🔬 Experimental Methodology & Task Design

### Reciprocal Tapping Task
Participants complete a 2D reciprocal tapping task based on Fitts' Law:
1. Two circular targets appear on a logical $1200 \times 600\text{ px}$ canvas.
2. One target is highlighted in red. Selecting it transfers the highlight to the opposing target and resets the timing interval.
3. Each distance/width combination ("card") requires **10 valid hits** to complete. Misses are logged as errors but do not decrement the hit counter.
4. The first click on each new target pair initiates the timer and is discarded from movement time calculations.

### Experimental Design
* **Design:** Within-Subjects ($1\times 2$ conditions: Mouse vs. Smartphone Tilt).
* **Counterbalancing:** Automatic assignment based on Participant ID (even codes start with Phone Tilt, odd codes start with Mouse).
* **No Hover Feedback:** To ensure visual parity across input modalities, targets do not light up upon cursor hover.

### Conditions & Index of Difficulty ($ID$)
The experiment evaluates 9 unique combinations of Distance ($A$) and Target Width ($W$), yielding difficulty indices ranging from **2.04 to 5.52 bits**:

$$\text{ID} = \log_2\left(\frac{A}{W} + 1\right)$$

| Parameter | Values |
| :--- | :--- |
| **Distances ($A$)** | 250 px, 500 px, 900 px |
| **Target Widths ($W$)** | 20 px, 40 px, 80 px |
| **Hits per Set** | 10 valid hits (plus initial start tap) |
| **Rounds (Blocks)** | 1 block per device (90 recorded hits per device + errors) |
| **Practice Phase** | 2 sets ($A=500, W=80$ and $A=900, W=40$) $\times$ 3 hits (unrecorded) |

---

## 🛠 Tech Stack & Architecture

The application is built as a self-contained, client-side web application designed for zero-install remote administration:

* **Frontend:** Single-file HTML5, CSS3, JavaScript (ES6+).
* **Real-time Remote Control:** [PeerJS](https://peerjs.com/) (WebRTC) establishes a peer-to-peer data channel between the participant's smartphone and desktop browser without requiring a intermediate server.
* **Device Motion API:** Captures mobile gyroscope orientation (`beta` and `gamma` tilt angles) in real time.
* **QR Code Generator:** Dynamically creates a WebRTC connection URL rendered via standard SVG.
* **Backend & Data Storage:** Google Apps Script (`/exec` web application endpoint) linked to a private Google Sheet with dynamic lock handling (`LockService`). Backup CSV generation is built directly into the client in case network requests fail.

---

## 📱 Smartphone Motion Processing

1. **Mapping:**
   * Horizontal position ($X$) maps directly to phone roll ($\gamma$, $\pm 30^\circ$ range).
   * Vertical position ($Y$) maps to phone pitch ($\beta$, $\pm 17.5^\circ$ range).
2. **Smoothing:**
   Frame-rate independent exponential interpolation (`lerp`) eliminates jitter caused by minor hand tremors:
   $$\text{Factor} = 1 - (1 - 0.25)^{\frac{\Delta t}{16.667\text{ ms}}}$$
3. **Selection Mechanism:**
   Tapping anywhere on the smartphone display triggers a target selection event (`pointerdown`).

---

## 📊 Recorded Data Parameters

Every trial writes a record containing the following variables:

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `participant` | String | Unique participant identifier (e.g., `P8412`). |
| `device` | String | Condition identifier (`mouse` or `phone`). |
| `input` | String | Hardware subtype (`mouse`, `trackpad`, or `tilt`). |
| `block` | Integer | Round sequence number. |
| `trial` | Integer | Absolute selection attempt counter. |
| `A` | Float | Target center-to-center distance (px). |
| `W` | Float | Target diameter (px). |
| `ID` | Float | Index of Difficulty calculated in bits. |
| `MT_ms` | Integer | Movement Time elapsed since previous successful selection (ms). |
| `hit` | Binary | Selection accuracy (`1` = Hit, `0` = Miss). |
| `clickX`, `clickY` | Float | Exact cursor coordinates upon selection event. |
| `targetX` | Float | Center $X$-coordinate of the intended target. |
| `scale` | Float | Scaling factor applied to adapt the $1200 \times 600$ canvas to display height/width. |
| `run` | String | Unique session execution token. |

---

## 🚀 How to Participate / Run Remotely

### Prerequisites
* A computer (desktop/laptop) with a modern web browser.
* A smartphone equipped with motion sensors and connected to the **same local network or public internet connection** as the computer.

### Steps
1. Open the hosted web application URL (via GitHub Pages).
2. Read the consent form and select **"Start"**.
3. Copy or save your generated Participant Code (e.g., `P1234`).
4. Select the designated starting condition (**Mouse** or **Phone tilt**).
5. **For Phone Tilt:**
   * Scan the displayed QR code using your smartphone camera.
   * Tap **"Turn on motion"** and grant orientation permissions.
   * Hold the smartphone flat in front of you and tap **"Set center"** to calibrate zero position.
   * Complete the practice round followed by the main evaluation block.
6. **Data Submission:**
   Results are uploaded automatically to the central research database. If an internet interruption occurs, use the **Download Results** button on the final screen to download your `.csv` file and email it to the experiment administrator.

---

## ⚙️ Configuration & Deployment Setup

To host your own deployment of this study:

1. **Google Apps Script Setup:**
   * Create a Google Sheet named `data`.
   * Open **Extensions > Apps Script** and deploy the `doPost` / `doGet` receiver script.
   * Set deployment permissions: **Execute as: Me**, **Who has access: Anyone**.
   * Copy the public URL ending in `/exec`.

2. **Frontend Configuration:**
   Edit the `CONFIG` object at the top of `index.html`:
   ```javascript
   const CONFIG = {
     contact: "your.email@domain.com",
     endpoint: "https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec"
   };
   ```

3. **Hosting:**
   Host `index.html` on any static web platform providing HTTPS (such as **GitHub Pages**), as mobile browsers restrict `DeviceOrientation` access on unencrypted HTTP connections.



   **Built this browser version with AI assistance on top of the original Processing sketch, to enable remote participation in the study.**