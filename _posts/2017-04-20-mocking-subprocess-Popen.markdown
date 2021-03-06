---
layout:     post
title:      "My Adventures with PYtest"
subtitle:   "Mocking"
date:       2017-04-20 18:00:00
author:     "az"
tags:       python
---
<p>
The other day I was writing a unit tests for an application that I support and spent more time than I should of, mocking out the subprocess module. So I thought I would write a post on this, in the event that I forget.
</p>

<p>
In the example, below, I had calls out to /bin/ip using subprocess.Popen, a linux command to return the IP addresses that are available on this system.

Lets get started, lets mock the subprocess call in a test fixture.

{% highlight python %}
import pytest

@pytest.fixture
def set_custom_ip_output(monkeypatch):
    def mock_return(command, shell=None, stdout=None ,stderr=None):
        return ip_popen_mocked()
        
    monkeypatch.setattr(subprocess, 'Popen', mock_return)
{% endhighlight %}

In the above example I have <a href="https://en.wikipedia.org/wiki/Monkey_patch">monkey patched</a> the subprocess.Popen call.
</p>
<p>
When Popen is now called and the fixture is imported, we will return the object ip_popen_mocked, lets now construct this object

{% highlight python %}
class ip_popen_mocked(str):
    """ Create a new string object the mimics subprocess.Popen behaviour by supporting
        .communicate method (probably a better way of doing this)
    """
    sbin_ip_output = '1: %(interface)s: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP \n    link/ether 9c:3b:fa:ea:23:36 brd ff:ff:ff:ff:ff:ff\n    inet %(ip_address)s brd %(broadcast)s scope global %(interface)s\n' # noqa

    def __init__(self):
        self.interface = 'bond0'
        self.ip_address = '192.168.0.1/24'
        self.broadcast = '192.169.0.255/24'

    def communicate(self):
        interface = self.interface
        ip_address = self.ip_address
        broadcast = self.broadcast
        return (sbin_ip_output % locals(), '')

        return (output, '')
{% endhighlight %}
I had to also include the method communicate, as after running Popen, i run .communicate against it
</p>
