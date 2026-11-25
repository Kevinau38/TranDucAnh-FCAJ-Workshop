---
title : "Reserve the volume and back up"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Before `prod-postgres` writes anything to the shared volume, it takes a **Write Exclusive** reservation. From that moment, the volume accepts writes only from `prod-postgres`, even though `test-postgres` is attached to the same volume.
