# Intermediate Data Examples from PAAB PDF Processing #

Based on the provided JSON example, here's how the data would look at different stages of processing:
1. Raw Node Example
A single text node from the PDF JSON output from pdf2json:

```json
{
  "x": 4.106,
  "y": 19.64,
  "w": 474.54,
  "clr": 0,
  "sw": 0.36196875,
  "A": "left",
  "R": [{
    "T": "Page%202.%20Preliminary%20comment.%20Please%20note%20that%20the%20header%20%22discover%20the%20power%20of%22%20must%20be%20restricted%20to%20the",
    "S": -1,
    "TS": [0, 13, 0, 0]
  }]
}
```

2. Processed Node (after getNode.js)

```json
{
  x: 4.106,
  y: 19.64,
  oc: undefined,
  url: undefined,
  text: "Page 2. Preliminary comment. Please note that the header \"discover the power of\" must be restricted to the",
  bold: 0,
  italic: 0,
  fontSize: 13,
  fontFaceID: 0,
  styleIndex: -1
}
```

3. Header Information (from getHeader.js)

```json
{
  addressLineOne: "Pharmaceutical Advertising Advisory Board",
  addressLineTwo: "PO Box 70 STN Main, Pickering, Ontario L1V 2R2",
  eFileNum: 129764,
  compCode: "CP-353007",
  brand: "Invega Sustenna",
  projectName: "HCP - Paper-Detail Aid",
  date: "November 14, 2022"
}
```

4. Reviewer Information (from getReviewer.js)

```
"Andrei Rotarus"
```

5. Comment Y-Ranges (from findCommentNodes function in getComments.js)

For the first page:

```json
[
  [19.64, 23.034],  // Comment 1
  [23.034, 25.678], // Comment 2
  [25.678, 29.404], // Comment 3
  [29.404, 34.511], // Comment 4
  [34.511, 36.404], // Comment 5
  [36.404, 39.799], // Comment 6
  [39.799, 43]      // Comment 7
]
```

6. Raw Comment Text Nodes (for Comment 1). 

```json
[
  // First node
  {
    x: 4.106,
    y: 19.64,
    text: "Page 2. Preliminary comment. Please note that the header \"discover the power of\" must be restricted to the",
    // other properties...
  },
  // Second node
  {
    x: 4.106,
    y: 20.39,
    text: "indication. Because page 3 of the layout discusses pharmacokinetic (PK) elements, the header cannot be",
    // other properties...
  },
  // Third node
  {
    x: 4.106,
    y: 21.14,
    text: "overarching to include the PK presentation. As such, we suggest restricting the prominence of the header ",
    // other properties...
  },
  // Fourth node
  {
    x: 4.106,
    y: 21.89,
    text: "discover the power of",
    // other properties...
  },
  // Fifth node
  {
    x: 10.046,
    y: 21.89,
    text: "\" to the same prominence accorded to the subheads on page 3.",
    // other properties...
  }
]
```

7. Parsed Plain Text Comment (for Comment 1). Plain text output is used for testing the algorithm functions in unit tests.

```
Page 2. Preliminary comment. Please note that the header "discover the power of" must be restricted to the indication. Because page 3 of the layout discusses pharmacokinetic (PK) elements, the header cannot be overarching to include the PK presentation. As such, we suggest restricting the prominence of the header "discover the power of" to the same prominence accorded to the subheads on page 3.
```

8. Parsed HTML Comment (for Comment 1). HTML output is used for the final output in the application and is stored in the postgres database comments table.

```html
Page 2. Preliminary comment. Please note that the header "discover the power of" must be restricted to the indication. Because page 3 of the layout discusses pharmacokinetic (PK) elements, the header cannot be overarching to include the PK presentation. As such, we suggest restricting the prominence of the header "discover the power of" to the same prominence accorded to the subheads on page 3.
```

9. Bullet List Example (from Comment 3)

Raw nodes with bullet markers:

```json
[
  // Main text
  { text: "The first subhead and its associated copy must have the relevant study details...", y: 25.678 },
  // More text nodes...
  // Bullet marker
  { text: "µ", y: 29.404 },
  // Bullet content
  { text: "Because page 3 of this current APS has multiple clinical presentations...", y: 29.428 },
  // More bullet content...
  // Sub-bullet marker
  { text: "λ", y: 32.404 },
  // Sub-bullet content
  { text: "Same comments applies to the third and fourth subheads (+ associated copies).", y: 32.428 }
]
```

Processed HTML with bullets:

```html
The first subhead and its associated copy must have the relevant study details (presented as footnotes in this iteration) repositioned somewhere in closer proximity. For reference, please see eFile # 115769, page 5 of the layout, which displays the demonstrated rapid onset of action, with the relevant study details being presented as footnotes on the same page (rather than at the end of the APS). (s4.1)<br/><br/>
<ul>
  <li>Because page 3 of this current APS has multiple clinical presentations (from multiple studies), each presentation must have its study design in closer proximity. You may choose to present them in the same font size as a footnote.
    <ul>
      <li>Same comments applies to the third and fourth subheads (+ associated copies).</li>
    </ul>
  </li>
</ul>
```

10. Approval Information (from getApproval.js)

```

10. Final Structured Output

```json
{
  header: {
    addressLineOne: "Pharmaceutical Advertising Advisory Board",
    addressLineTwo: "PO Box 70 STN Main, Pickering, Ontario L1V 2R2",
    eFileNum: 129764,
    compCode: "CP-353007",
    brand: "Invega Sustenna",
    projectName: "HCP - Paper-Detail Aid",
    date: "November 14, 2022"
  },
  reviewer: "Andrei Rotarus",
  htmlComments: [
    "Page 2. Preliminary comment. Please note that the header \"discover the power of\" must be restricted to the...",
    "Page 3. Preliminary comment. We question the checkmarks beside each of the subheads. We do not accept...",
    // More HTML-formatted comments with bullet lists
  ],
  comments: [
    "Page 2. Preliminary comment. Please note that the header \"discover the power of\" must be restricted to the...",
    "Page 3. Preliminary comment. We question the checkmarks beside each of the subheads. We do not accept...",
    // More plain text comments
  ],
  approvalID: null // Since there are comments, no approval ID
}
```

This shows how the raw PDF JSON data is transformed through various processing steps into structured, formatted content that preserves the original document's meaning and organization while making it more useful for granular data storage of individual PAAB comments and more accessible for display and interaction within the application. 

This data extraction algorithm allows the application create a relational database of comments, responses, approvals, pushbacks, and other metadata that can be used to power the application's features. E.g. later we can use the comments to generate a report of all comments made by a reviewer, or a report of all comments made on a specific brand, or even as input to a machine learning model to predict the best response to a comment.