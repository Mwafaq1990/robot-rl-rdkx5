# Dofbot Ultra × RDK X5 — Vision-Guided RL Pick-and-Place

My project for the **Robotics Dream Keeper Challenge 2026 Winter** (D-Robotics).

**Goal:** A Yahboom Dofbot Ultra robotic arm that finds objects with YOLO running on the RDK X5 BPU, and grasps them with a reinforcement-learning (PPO) policy trained in MuJoCo simulation, all connected with ROS 2.

| Stage | Status | Folder |
|---|---|---|
| Stage 1: Ignite | ✅ Done | [`stage1/`](stage1/) |
| Stage 2: Build | ⏳ In progress | `stage2/` |
| Stage 3: Launch | ⏳ Planned | — |

---

## Hardware

- Yahboom **Dofbot Ultra** robotic arm
- D-Robotics **RDK X5** (8 GB RAM, 10 TOPS BPU)
- Depth camera (USB, appears as `/dev/video0` and `/dev/video1`)
- SanDisk Extreme 64 GB microSD (official RDK image for Stage 1)
- Development PC: Windows 11 + WSL2 Ubuntu 24.04

## Software

| Item | Version |
|---|---|
| Board OS | RDKOS 3.5.0 Desktop (Ubuntu 22.04.5, kernel 6.1.83, aarch64) |
| ROS 2 | TogetheROS.Bot (TROS), ROS 2 Humble |
| Flash tool | RDK Studio |
| AI model | YOLOv8 (`dnn_node_example`, BPU) |

---

## Stage 1: Ignite

### 1. Flash the system image
1. Put the microSD card into a USB card reader and connect it to the PC.
2. RDK Studio → **Open flash setup** → device **RDK X5** → image **RDKOS 3.5.0 Desktop**.
3. Check the target drive (about 64 GB, USB), then **Erase and flash**.
4. Eject the card, insert it into the RDK X5 and power on.

Evidence: [`stage1/01-flash-success.png`](stage1/01-flash-success.png)

### 2. Network and SSH
1. Connect the board to Wi-Fi from the desktop (network icon, top right).
2. Find the board's IP address on the board: `hostname -I`
3. From the PC (WSL Ubuntu):
   ```bash
   ssh sunrise@<BOARD_IP>
   ping -c 3 google.com
   ```

Evidence: [`stage1/02-ssh.png`](stage1/02-ssh.png)

### 3. Sensor: USB depth camera
```bash
ls /dev/video*
# /dev/video0  /dev/video1
```
Interface: **USB** (UVC camera).

Evidence: [`stage1/03-camera.png`](stage1/03-camera.png)

### 4. First AI task: YOLOv8 object detection on the BPU
```bash
source /opt/tros/humble/setup.bash
cp -r /opt/tros/${TROS_DISTRO}/lib/dnn_node_example/config/ .
export CAM_TYPE=usb
ros2 launch dnn_node_example dnn_node_example.launch.py \
  dnn_example_config_file:=config/yolov8workconfig.json \
  dnn_example_image_width:=480 dnn_example_image_height:=272
```
View the live result in a browser on the PC: `http://<BOARD_IP>:8000`
Stop with `Ctrl + C`.

Evidence: [`stage1/04-yolo.png`](stage1/04-yolo.png)

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Windows can't see the SD card after flashing | Normal: the card uses Linux partitions. Click **Cancel** if Windows asks to format it. |
| "Translation unavailable" lines in the RDK Studio log | Only the UI translation feature; flashing is not affected. |
| No free USB port for a keyboard on the Dofbot | Temporarily unplug the camera USB, set up Wi-Fi, then plug the camera back in. |
| Camera shows two devices (`video0`, `video1`) | Normal for a depth camera (color + depth stream). |
| SSH password prompt shows nothing while typing | Normal in Linux; type the password and press Enter. |

---

## Roadmap

- **Stage 2 (by Nov 26):** proposal, architecture, ROS 2 graph, compute plan, risks
- **Stage 3 (by Dec 1):** MuJoCo sim + PPO training → ONNX policy on the board, YOLO on BPU, ROS 2 integration, safety limits + e-stop, benchmark, demo video

## Credits and license

- D-Robotics RDK X5, TogetheROS.Bot and `dnn_node_example`
- Yahboom Dofbot Ultra hardware and example code — [github.com/YahboomTechnology/DOFBOT-Ultra](https://github.com/YahboomTechnology/DOFBOT-Ultra), MIT License, Copyright (c) 2020 Shenzhen Yahboom Tech. Any Yahboom files used here are kept in [`third_party/yahboom/`](third_party/yahboom/) together with their original license.
- My own code in this repository: MIT License
