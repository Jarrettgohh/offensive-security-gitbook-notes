# Introduction

_Hello readers!_

This Gitbook document summarizes my findings and learnings on various offensive security concepts, such as penetration testing, ethical hacking and red teaming, based on my knowledge and research. While I strive for accuracy, some notes may be incomplete or incorrect. I'm continuously updating and refining the content as I learn and gain new insights.

If you notice any mistakes or have suggestions for improvement, feel free to contact me at [jarrettgxz.sec@gmail.com](mailto:jarrettgxz.sec@gmail.com). Your feedback is greatly appreciated, thank you!

Without further ado, let's hack away!

#### Note on shell prompt convention <a href="#shell-prompt-convention" id="shell-prompt-convention"></a>

The `$` symbol will be used to indicate a shell prompt (defaulting to _Bash_). Some commands may require _root_ privileges via the `sudo` command. However, this documentation will not explicitly specify when `sudo` is required — it should be prepended where necessary.

#### Note on use of generative AI <a href="#shell-prompt-convention" id="shell-prompt-convention"></a>

> This document was supported by OpenAI's ChatGPT, which was used to generate ideas, provide validation, and clarify technical concepts.&#x20;
>
>
>
> However, most of the core theories and principles discussed in this documentation were developed and tested by me, based on research and experimentation conducted to the best of my abilities.



_I have received my inspiration and knowledge hugely from the content provided by the following sources:_

1. **Hacktricks**

{% embed url="https://book.hacktricks.xyz/" %}

2. **TryHackMe**

{% embed url="https://tryhackme.com" %}

### Shell prompt convention

The `$` symbol will be used to indicate a shell prompt (defaulting to _Bash_). Some commands may require _root_ privileges via the `sudo` command. However, this documentation will not explicitly specify when `sudo` is required —  it should be prepended where necessary.
