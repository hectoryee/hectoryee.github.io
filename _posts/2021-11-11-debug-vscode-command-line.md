---
title: Debug Python Module on VS Code from Command Line
excerpt: Learn debugging on VS code. Not responding on VS code terminal so run debug from command line.
published: true
date: 2021-11-11
categories: project
tags: bash docker
classes: wide
header:
  image: /assets/images/2021-11-11.png
  image_description: "A description of the image"

---

Follow instructions from [VS Code website](https://code.visualstudio.com/docs/python/debugging#_command-line-debugging).

Configure `launch.json` to attach process from command line.

``` json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Attach",
            "type": "python",
            "request": "attach",
            "connect": {
              "host": "localhost",
              "port": 5678
            }
          }
        // {
        //     "name": "Python: Module",
        //     "type": "python",
        //     "request": "launch",
        //     "module": "main.py",
        //     "args": ["--config", "config/ted/asr_rnnext.yaml", "--njobs", "8", "--load", "ckpt/asr_rnnext_sd0/best_ctc.pth", "|", "tee", "-a", "result/$(date +%F)-rnnext.txt"]
        // }
    ]
}
```

Run python command as usual. `python -m debugpy --listen 5678 main.py --config config/ted/asr_rnnext.yaml  --njobs 8 --load ckpt/asr_rnnext_sd0/best_ctc.pth | tee -a result/$(date +%F)-rnnext.txt`

`python -m debugpy --listen 5678 script.py`.