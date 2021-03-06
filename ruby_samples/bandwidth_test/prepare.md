# PREPARE FOR BANDWIDTH TEST

This part is only required for setup / preparation of the bandwidth test tool.

## A. PREPARE SERVER & CLIENT

   1. Install Ruby at each side

      ```bash
      $ sudo apt-add-repository ppa:brightbox/ruby-ng
      $ sudo apt-get update
      $ sudo apt-get install ruby2.2 ruby2.2-dev
      ```

   2. Configure the server EMS to allow remote Telnet.
      In `/etc/evostreamms/config.lua`, under `CLI acceptors`, for protocol `inboundJsonCli`, change `ip` from `127.0.0.1` to `0.0.0.0`:

      ```lua
      -- CLI acceptors
      {
              ip="0.0.0.0",
              port=1112,
              protocol="inboundJsonCli",
              useLengthPadding=true
      } 
      ```

   3. Check if the EMS is online by browsing:

      ```text
      http://<ems_ip>:8888/EMS_Web_UI/index.php
      ```

      then clicking Connect. Perform this for each side.

## B. PREPARE FOR RTMP STREAMING

### CLIENT SIDE

   - If the EMS is not running at the client, start it.

### SERVER SIDE

   1. If the EMS is not running at the server, start it.

   2. Copy a 1-minute video, `sintel1m720p.mp4`, and 2 text files from the GitHub:

      ```bash
      $ wget http://docs.evostream.com/sample_content/assets/sintel1m720p.mp4
      $ wget http://docs.evostream.com/sample_content/assets/sintel1h.sh
      $ wget http://docs.evostream.com/sample_content/assets/sintel1h.list
      ```

   3. Install `ffmpeg` if required

      ```bash
      $ sudo apt-add-repository ppa:mc3man/trusty-media
      $ sudo apt-get update
      $ sudo apt-get install ffmpeg
      ```

   4. Run the script `sintel1h.sh` to create a 1-hour video, `sintel1h720p.mp4`.

      ```bash
      $ sh sintel1h.sh
      ```

   5. Move the 1-hour video, `sintel1h720p.mp4`, to the EMS media folder:

      ```bash
      $ sudo mv sintel1h720p.mp4 /var/evostreamms/media
      ```

   6. Change ownership and permissions to allow MP4 playback:

      ```bash
      $ cd /var/evostreamms/media
      $ sudo chown evostreamd:evostreamd sintel1h720p.mp4
      $ sudo chmod a+r sintel1h720p.mp4
      ```

   7. Test the playability of the MP4 test file, typically:

      ```bash
      $ ffplay rtmp://<ems_ip>/vod/sintel1h720p.mp4
      ```

      Alternatively, use VLC, JWPlayer or some other Flash player to play it back.

   8. If the MP4 test file is not playable, review steps A-3 and B-6.
   
  
## C. SETUP PARAMETERS

   Modify the following parameters in `pullscript.rb` before running tests.

   1. *REMOTE_IP* = partner IP address
   2. *IN_URI* = source URI for pullstream

      The sample video, `sintel1h720p.mp4`, has a bandwidth of `1.165 megabites/sec`.

   3. *IN_STREAM_NAME* = localstreamname for pullstream

   4. *RAMP_STEPS* = number of steps to ramp up/down (see calculations below)

   5. *RAMP_PULL_COUNT* = number of additional streams to pull per step (see calculations below)

   6. *RAMP_STEP_DELAY* = delay in seconds between steps

   7. *RAMP_PEAK_TIMES* = number of delay loops after ramp-up before ramp-down

### Note

   Other settings that can limit RTMP performance for Linux:

   ```bash
   $ ulimit -n
   ```

   Set `ulimit` in excess of the number of expected streams. Refer to the OS documentation on how to set `ulimit`.


## D. CALCULATIONS

   The maximum number of streams will be:

   ```
   Maximum_Streams = RAMP_STEPS x RAMP_PULL_COUNT
   ```
   
   The maximum bandwidth will be:

   ```
   Maximum_Bandwidth = Maximum_Streams x Source_Bandwidth
                     = RAMP_STEPS x RAMP_PULL_COUNT x Source_Bandwidth
   ```

   For example, with *RAMP_STEPS* = 10, *RAMP_PULL_COUNT* = 8, and a source bandwidth of 1.165 megabits/sec:

   ```
   Maximum_Streams = 10 x 8 = 80 streams
   Maximum_Bandwidth = 80 x 1.165 megabits/sec = 93.2 megabits/sec
   ```

   Ramping up further would saturate a 100-baseT network, but would use only around 10% of a gigabit network's bandwidth.

   The maximum duration will be:

   ```
   Maximum_Duration = (RAMP_STEPS x 2 + RAMP_PEAK_TIMES) x RAMP_STEP_DELAY

   ```
   For, with *RAMP_STEPS* = 10, *RAMP_PEAK_TIMES* = 5 and *RAMP_STEP_DELAY* = 20

   ```
   Maximum_Duration = (10 x 2 + 5) x 20 = 500 seconds
   ```

   The source stream should have a duration longer than this to be able to ramp up/down smoothly with the expected bandwidth profile.

