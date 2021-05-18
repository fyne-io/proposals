# Material Design "Density"

Material Design speaks a lot about "Density" throughout the specification, which I feel they use to refer to the amount of clearspace between similar elements within the same control. (Example: List template within the greater List, Table cells within the greater Table, etc.

Please consider the following reference with a grain of salt, as even Material Design has marked this part of the specification "Beta". That being said, they have applied this "Beta" to all of their released products, so I would argue "Google feels it is already production-ready."

https://material.io/design/layout/applying-density.html

## Background

I feel because `fyne` currently **defaults** to "Dense" layouts as described by Material Design, we are choosing to break the specification in a number of ways. It could be construed/argued as to whether this specification speaks of margins or paddings, but `fyne` appears to only have one concept between these two, so I feel the distinction is moot for now.

## Architecture / API

[Details of any changes to API or architecture]

## Design

[Screenshots or description of graphical elements]

## Implementation

[Additional details that would be required, such as hardware, OS specifics or external APIs]

## Prior art

### Google Drive

<table cellpadding=0 cellspacing=0 border=0 width="100%"><tr>
  <td><img src="https://imgur.com/fRWG3Jt.png" alt="Google Drive Dense Layout"/></td>
  <td><img src="https://imgur.com/Uf3Lftn.png" alt="Google Drive Comfortable Layout"/></td>
  </tr><tr><td>Current Default (and only option): "Compact"</td>
  <td>Should be an option: "Comfortable"</td></tr></table>

### Google Mail

<table cellpadding=0 cellspacing=0 border=0 width="100%"><tr>
  <td><img src="https://4.bp.blogspot.com/-e9GKhDERJJ4/TrgYWF_BaFI/AAAAAAAAAP4/wgXPYS_KDQg/s1600/compact.png" alt="GMail Dense Layout"/></td>
  <td><img src="https://2.bp.blogspot.com/-GX9uo0LibAg/TrgYWQPqhWI/AAAAAAAAAQI/ApY3ARfv-rU/s1600/newlarge.png" alt="GMail Comfortable Layout"/></td>
  </tr><tr><td>Current Default (and only option): "Compact"</td>
  <td>Should be an option: "Comfortable"</td></tr></table>
