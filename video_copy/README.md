# Copy WebSocket Stream to ZLMediaKit RTSP Server

## Run Zlmediakit

```bash
sudo docker run -d --restart unless-stopped -id -p 1935:1935 -p 8080:80 -p 8554:554 -p 10000:10000 -p 10000:10000/udp -p 8000:8000/udp -p 30000-30500:30000-30500 -p 30000-30500:30000-30500/udp --name zlmediakit --env MODE=standalone -e TZ="Asia/Shanghai" zlmediakit/zlmediakit:master

docker logs zlmediakit -f  # to view logs
```

## Access Zlmediakit Web UI
Open web browser and go to http://<jetson-ip>:8080


## RTSP Stream URL
rtsp://<jetson-ip>:8554/live/nanoowl

## Push Websocket Stream to RTSP Server
Create main.py file

```python
import asyncio
import websockets
import subprocess
import shlex

# The URL of your WebSocket stream
WEBSOCKET_URL = "ws://localhost:7860/ws"

# The URL where ZLMediaKit will receive the stream
ZLMEDIAKIT_RTSP_URL = "rtsp://localhost:8554/live/nanoowl"

async def websocket_to_rtsp():
    print(f"Connecting to WebSocket at {WEBSOCKET_URL}")

    # The FFmpeg command to push H.264 data to ZLMediaKit
    # Assuming the WebSocket sends raw H.264 video. Adjust if different.
    ffmpeg_cmd = (
        f"ffmpeg -i - "  # Input from stdin
        f"-c:v copy "    # Copy video codec without re-encoding
        f"-c:a copy "    # Copy audio codec (if any)
        f"-f rtsp -rtsp_transport tcp {ZLMEDIAKIT_RTSP_URL}"
    )

    # Use subprocess.Popen to run FFmpeg and pipe stdin
    ffmpeg_proc = subprocess.Popen(shlex.split(ffmpeg_cmd), stdin=subprocess.PIPE, shell=False)

    try:
        async with websockets.connect(WEBSOCKET_URL) as ws:
            print("WebSocket connected. Starting stream push to ZLMediaKit.")
            await ws.send("prompt:[a face[a nose,an eye]]")
            while True:
                # Receive a message (frame) from the WebSocket
                frame = await ws.recv()
                
                # Write the frame to FFmpeg's standard input
                if isinstance(frame, bytes):
                    ffmpeg_proc.stdin.write(frame)
                    
    except websockets.exceptions.ConnectionClosed:
        print("WebSocket connection closed.")
    except ConnectionRefusedError:
        print("Could not connect to WebSocket. Is the server running?")
    finally:
        print("Terminating FFmpeg process.")
        ffmpeg_proc.stdin.close()
        ffmpeg_proc.wait()

if __name__ == '__main__':
    # Make sure to replace with your actual WebSocket and ZLMediaKit URLs
    asyncio.run(websocket_to_rtsp())

```

## Run main.py to push stream to ZLMediaKit

```bash
python3 main.py
```

## Access RTSP Stream with VLC 3.0.20+
Open VLC Media Player and go to Media -> Open Network Stream, then enter the RTSP URL:

```
rtsp://<jetson-ip>:8554/live/nanoowl
```



## Create docker image
```bash
sudo docker build . -t dustynv/vcopy:1.0.0
```

## Run Zlmediakit

```bash
sudo docker run -d --restart unless-stopped -id --network host --name video_copy --env MODE=standalone -e TZ="Asia/Shanghai" dustynv/vcopy:1.0.0

docker logs video_copy -f  # to view logs
```



## Install as a Systemd Service
```bash
sudo chmod +x /home/unitree/video_copy/main.py
sudo cp /home/unitree/video_copy/vcopy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable vcopy.service
sudo systemctl start vcopy.service
```