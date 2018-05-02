# RTSP/RTP server in python #

Forked from [Video-Streaming-Server-and-Client](https://github.com/TibbersDriveMustang/Video-Streaming-Server-and-Client).
I am trying to make it completely compatible with real RTSP clients, like VLC or ffplay

Requirements:
- python 3.5. I use a lot of `yield from` in protocol FSMs
- tornado, to spin my lovely coroutines and do socket stuff
- pil/pillow
- tkinter
- pyzmq - for local communication between RTSP/RTP server and data provider

How to:

# Running #

    Open a terminal:
        python server_main.py 1025

    Open another terminal:
        python ClientLauncher.py 127.0.0.1 1025 5008 video.mjpeg

Start the server with the command line
	
		python server_main.py server_port
	
	Where server_port is the port your server listens to for incoming RTSP connections
	    # 1025
		# Standard RTSP port is 554 
		# But need to choose a #port > 1024

Open a new terminal

ffmpeg:
`ffplay -loglevel debug -i rtsp://localhost:1025/video.mjpeg`

live555 testRTSPClient (used in VLC):
`./testProgs/testRTSPClient rtsp://localhost:1025/video.mjpeg`

VLC:
`vlc --verbose=1 --file-logging --logfile=vlc-log.txt rtsp://localhost:1025/video.mjpeg`

	Start the client with the command line

		python ClientLauncher.py server_host server_port RTP_port video_file

	Where 
		# server_host : the name of the machine where server is running (here "127.0.0.1")
            Use command line
                hostname
            to get the hostname(IP address,human readable hostname may not work)
		# server_port : port the server is listening on (here "1025")
		# RTP_port : port where the RTP packets are received (here "5008")
		# video_file : name of video file you want to request,here "video.mjpeg"
	

		@ file format
			Lab`s` proprietary MJPEG(Motion JPEG) format
				# The server streams a video which has been encoded into a proprietary MJPEG file format
				# This format stores the video as concatenated JPEG-encoded images
				# Each image being preceded by a 5-Byte header which indicates the bit size of the image
				# Server parses the bitstream of MJPEG file to extract the JPEG images
				# Server sends the images to client at periodic intervals
				# Client then displays the individual JPEG images sent from server			

        @ Client 
            
                # Send RTSP commands to server by pressing buttons:
                    > RTSP(Real Time Streaming Protocol) 
                        * A network control protocol designed for use in entertainment and communications system to control streaming media servers
                        * Used for establishing and controlling media sessions between end points
                        * Clients issue VCR-style commands e.g play, pause .. to facilitate real-time control of playback of media files from server
                # Most RTSP servers use the RTP(Real-time Transport Protocol) in conjunction with RTCP(Real-time Control Protocol) for media stream delivery.
        
                # Commands
                    
                    > SETUP
                        * Send SETUP request to the server
                        * Insert Transport header(specify port for RTP data socket you just created)
                        * RTP : Real-time Transport Protocol
                        * Read server`s` response
                        * Parse Session header(from response) to get RTSP Session ID
                        * Create a datagram socket for receiving RTP data
                        * Set timeout on socket to 0.5 seconds
                
                    > PLAY
                        * Send PLAY request
                        * Insert Session header
                        * Use the Session ID(returned in the SETUP response)
                        * Not put the Transport header in the request
                        * Read the Server`s` response

                    > PAUSE 
                        * Send PAUSE request
                        * Insert the Session header
                        * Use the Session ID returned in the SETUP response
                        * Not put the Transport header in this request
                        * Read the server`s` response

                    > TEARDOWN
                        * Send TEARDOWN request
                        * Insert the Session header
                        * Use the Session ID returned in the SETUP response
                        * Not put the Transport header in this request
                        * Read the server`s` response

                    *** Must insert CSeq header in every request you send
                            Which starts at 1 and incremented by one for each request you send
