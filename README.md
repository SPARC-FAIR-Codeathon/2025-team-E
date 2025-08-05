# 2025-team-E

# [Toxic Trace](https://team-e-toxic-trace-50672fa229b6.herokuapp.com/)

<img width="3577" height="1886" alt="image" src="https://github.com/user-attachments/assets/60b42fc8-152c-4e76-8e21-11d3c42082a2" />


Our main submission is Toxic Trace, a platfrom for exporing the effect of toxins on the human body.

It does this by:
 - Providing info on how toxins effect the body
 - Quizing users to create a health score for their organs
 - Using a 3d molecule explorer for how toxins bind
 - A filterable database for toxins and effects

The website can be viewed [here](https://team-e-toxic-trace-50672fa229b6.herokuapp.com/)
Code for the site can be viewed [here](https://github.com/SPARC-FAIR-Codeathon/2025-team-E/tree/main/toxic-trace-web-app)

The website is a [git](https://github.com/ABI-Software/mapintegratedvuer) fork, so can be easily deployed to sparc.science (and other websites)


## Molecule Viewer Documentation

# Molecular Structure Viewer

A web-based 3D molecular visualization tool for exploring protein-ligand interactions, built with 3Dmol.js.

This molecular viewer was developed to visualize and analyze the binding interactions between environmental toxins and biological receptors.

## Current Molecular Systems

The viewer showcases three examples of how environmental toxins interfere with human endocrine and metabolic systems by competing with natural hormones and signaling molecules for receptor binding sites.

1. **Weak Agonism** (BPA → ER-α): Even weak binding can cause biological effects when exposure is chronic
2. **Strong Agonism** (PFOA → PPAR-γ): Some toxins bind more strongly than natural ligands, causing overactivation
3. **Competitive Inhibition** (PFOS → Transthyretin): Toxins can interfere with hormone transport and availability

Together, they demonstrate why environmental toxins pose significant health risks even at relatively low concentrations, and highlight the importance of understanding molecular-level interactions in toxicology research.

### 1\. Estrogen Receptor α (ER-α LBD) System

- **Receptor**: Estrogen Receptor α Ligand Binding Domain
- **Natural Ligand**: 17-β-estradiol (the primary female sex hormone)
- **Toxin**: Bisphenol A (BPA)
- **Significance**: BPA, a plastic additive ubiquitous in thermal receipts and polycarbonate bottles, acts as an endocrine disruptor by binding to the estrogen receptor. However, it binds approximately 6,000-fold weaker than the natural hormone, meaning higher concentrations are needed to elicit biological effects. This weak binding still poses health risks due to the widespread exposure to BPA in modern environments.
- **PDB References**: BPA complex (3ERT), Estradiol complex (1A52)

### 2\. PPAR-γ Nuclear Receptor System

- **Receptor**: Peroxisome Proliferator-Activated Receptor gamma (PPAR-γ)
- **Natural Ligand**: Long-chain fatty acids
- **Toxin**: Perfluorooctanoic acid (PFOA)
- **Significance**: PFOA, a persistent "forever chemical" used in non-stick coatings and water-resistant materials, demonstrates remarkably strong binding to PPAR-γ. Surprisingly, it binds approximately 55-times more tightly than the receptor's natural fatty acid partners. This enhanced binding disrupts normal metabolic regulation, potentially contributing to obesity, diabetes, and other metabolic disorders associated with PFAS exposure.
- **PDB References**: PFOA complex (8U57), Fatty acid complex (2ZK1)

### 3\. Transthyretin Transport Protein System

- **Receptor**: Transthyretin (thyroid hormone transport protein)
- **Natural Ligand**: Thyroxine (T4, thyroid hormone)
- **Toxin**: Perfluorooctane-sulfonic acid (PFOS)
- **Significance**: PFOS, another persistent environmental pollutant from the PFAS family, interferes with thyroid hormone transport by binding to transthyretin. It binds approximately 4-times weaker than the natural thyroid hormone thyroxine. Despite this weaker binding, PFOS can disrupt thyroid hormone homeostasis, potentially leading to developmental, metabolic, and neurological effects, particularly concerning given the critical role of thyroid hormones in human development.
- **PDB References**: PFOS complex (5JID), T4 complex (1ICT)

## User Interface Guide

### Controls Panel

- **Toggle Button** (☰): Show/hide the sidebar
- **Structure List**: View all loaded molecules with individual controls
- **Color Picker**: Change the color of each structure
- **Visibility Toggle** (👁): Show/hide individual structures
- **Center Button** (⊕): Focus view on a specific molecule

### Global Controls

- **Reset View**: Return to default zoom and position
- **Center All Visible**: Fit all visible structures in view
- **Hide All**: Hide all structures
- **Show All**: Show all structures

### Mouse Controls

- **Left Click + Drag**: Rotate the view
- **Right Click + Drag**: Translate the view
- **Scroll Wheel**: Zoom in/out
- **Middle Click + Drag**: Zoom (alternative)

## Technical Details

### Built With

- **3Dmol.js**: A modern, WebGL-based molecular viewer
- **JavaScript**: No framework dependencies
- **HTML5 & CSS3**: For structure and styling

## Features

- **Interactive 3D Visualization**: Rotate, zoom, and pan molecular structures in real-time
- **Multi-Structure Support**: Load and display multiple proteins and ligands simultaneously
- **Customizable Display**:

  - Proteins rendered as cartoon representations
  - Ligands displayed as stick models
  - Adjustable colors for each structure; natural ligand in green and receptor in red

- **Structure Management**:

  - Toggle visibility of individual structures
  - Center view on specific molecules
  - Show/hide all structures with one click

- **Responsive Design**: Collapsible sidebar for maximum viewing area

## Installation & Usage

### Quick Start

1. Download the HTML file
2. Star a server and open it in a web browser (server required)

### Integration into Web Applications

The viewer is designed to be easily integrated into larger web applications:

```html
<!-- Simply include the HTML file in your project -->
<iframe src="molecular_viewer.html" width="100%" height="600px"></iframe>
```

Or embed directly into your application's structure.

### Customizing Molecular Data

To add your own molecular structures:

1. Locate the `PDB_FILES` object in the HTML file
2. Replace the file and names keys with your PDB files and names:

```javascript
const PDB_FILES = [
  { name: "Peroxisome Receptor", file: "2zk1_receptor.pdb", color: "#708090" },
  { name: "Natural (Fatty Acid)", file: "2zk1_ligand.pdb", color: "#32CD32" },
  { name: "Toxin (PFOA)", file: "8u57_ligand.pdb", color: "#FF6347" },
];
```

1. Locate the `help-tooltip` element in the HTML file
2. Replace the title ('PPAR-γ nuclear receptor'), description ('Perfluorooctanoic acid (PFOA)...') and citation ('PDB templates: PFOA in 8U57; Fatty Acid in 2ZK1').

```javascript
<div class="info-panel" id="infoPanel" style="display: none;">
  <h4>PPAR-γ nuclear receptor</h4>
  <div class="stats">
    <p id="structureCount"></p>
    <p id="totalAtoms"></p>
    <p id="visibleStructures"></p>
  </div>
  <div class="description">Perfluorooctanoic acid (PFOA)...</div>
  <div class="citation">(PDB templates: PFOA in 8U57; Fatty Acid in 2ZK1)</div>
</div>
```

## *Disclaimer*

A few of the links in our database are going to the wrong papers unfortunately, so if the paper you end up in looks wrong, it probably is
# FundLab
