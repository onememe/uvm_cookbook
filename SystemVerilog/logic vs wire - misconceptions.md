
## implicit behaviours in SV - be cautious!

```systemverilog
/** 
 * In SystemVerilog an internal signal or port can be
 * declared implicitly or explicitly. Typical explicit 
 * declaration looks like:
 */
// port declaration
<direction> <net or var type> <data type> <port name>

// internal signal declaration
<net or variable> <data type> <signal name>

/**
 * SysetmVerilog allows to skip one or more declaration 
 * parameters. This leads to implicit behaviors as 
 * decribed in detail in the SV standard LRM.
 *
 * 1. If direction is omitted, it defaults to <inout>
 * 2. If net/variable type is omitted for <input> port
 *    it defaults to net type: wire
 * 3. If net/variable type is omitted for an <output>, 
 *    it depends on the data type
 *     - if it's omitted as well, id defaults to net
 *       type: wire
 *     - else it defaults to variable type
 * 4. If data type is omitted, it defaults to logic
 * 5. If net/variable type is omitted for an internal
 *    signal and is not used in port connections, it
 *    defaults to variable type.
 */
// case 1:
output logic[3:0] data;  // eq to output var logic[3:0]
...
assign data = 1'z;
assign data = a;
// will cause an error

// case 2:
output wire[3:0] data;  // eq to output wire logic[3:0]
...
assign data = 1'z;
assign data = a;
// it's ok.
```