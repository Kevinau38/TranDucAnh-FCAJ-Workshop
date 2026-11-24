---
title : "Optimize IOPS"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


In this step you create 1,024 empty files on EFS in three different ways. Creating a file is a metadata operation, so this test measures IOPS rather than throughput.

1. Run the three commands. Each one creates its own random directory under `/efs/tutorial/touch`:

   ```bash
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}; time for i in {1..1024}; do touch /efs/tutorial/touch/${directory}/test-1.3-$i; done
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}; time seq 1 1024 | parallel --will-cite -j 128 touch /efs/tutorial/touch/${directory}/test-1.4-{}
   directory=$(echo $(uuidgen)| grep -o ".\{6\}$"); mkdir -p /efs/tutorial/touch/${directory}/{1..32}; time seq 1 32 | parallel --will-cite -j 32 touch /efs/tutorial/touch/${directory}/{}/test1.5{1..32}
   ```

   - (1) One `touch` at a time, in one directory: about 11 seconds.
   - (2) 128 `touch` commands at the same time, in one directory: about 6 seconds.
   - (3) 32 jobs, each creating 32 files in its own directory: under 1 second.

   ![Create 1,024 files in three ways](/images/4-efs/4.1-touch.png)

The third method is more than 15 times faster than the first. It runs many operations in parallel, spreads them over many directories so they do not compete for the same directory, and starts only 32 processes instead of 1,024. The best way to use EFS is to work with many threads and many directories at the same time.
