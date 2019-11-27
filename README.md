# encodewithx264-k8s

I use this script for encoding mythtv recordings inside of my r-pi kubernetes cluster.

It creates a kubernetes job that runs ffmpeg optimized for r-pi's, reads the source video from NFS and writes it to an NFS destination.

## Usage

Put this script somewhere. I like to put it in `/usr/local/bin/` but anywhere will do. Make sure it is executable.

Setup a [user job](https://www.mythtv.org/wiki/User_Jobs) in MythTV:

```
<path-to-script>/encodewithx264-k8s "%DIR%" "%FILE%" "%TITLE%" "%SEASON%" "%EPISODE%" "192.168.1.10:/mnt/dvr"  "192.168.1.11:/volume1/media" "tv"
```

The first 5 arguments come from MythTV substitutions. The last 3 you will need to fill in based on your specific details.

Argument 6 is the source NFS volume for where the job inside of Kubernetes will read the MythTV source file. You'll need to make sure that your MythTV server exports this directory via NFS `/etc/exports`.

Argument 7 is the destination NFS volume for where the job inside of Kubernetes will write the MKV to.

Arguments 6 and 7 can be, but doesn't have to the be the same server as the source. In my usage, the source is my Linux MythTV server and the destination is a Synology NAS.

Argument 8 is the path inside of the destination to write the file to. In the example above, it would be written to `/volume1/media/tv` on the destination NFS server's filesystem.

## Caveats / Notes

- This script relies on your MythTV install to properly record season/episode numbers.
- It restricts the jobs to run distinct on each kubernetes node. It's recommended to set your concurrent user job limit in MythTV to the same number of worker nodes in your kubernetes cluster. If you don't do this the jobs will build up in the cluster instead of in MythTV. They may generate errors instead? This author has never actually tried it!
- The quality configuration for ffmpeg is fairly high. It copies the audio steam as-is and sets the video bitrate to 5M/s. This comes out to about 2-2.5GB per hour for 1080p HDTV. This author isn't aware of any kind of dynamic bitrate option for the h264 codec in the r-pi ffmpeg program. Patches welcome here!
- Encoding an hour of HDTV on a r-pi 3 takes about 1.5 hours. My cluster is a 4 node cluster of r-pi 3's, but I suspect the new pi 4s could do this in much less time!
