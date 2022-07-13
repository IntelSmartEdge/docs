```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Reference Implementations
Intel created several example solutions to serve as reference implementations (RIs) for the Intel® Smart Edge infrastructure. These RIs address identified use cases for an edge cloud infrastructure, in different industry segments. Application developers in ISV or end-user organizations can install and study these RIs, and use them as a base for their own applications. 

A set of RIs have been created and hosted in the [Intel® Developer Catalog](https://www.intel.com/content/www/us/en/developer/tools/software-catalog/overview.html?s=Newest&q=%22smart%20edge%20open%22). To access these RIs, you will need to create an Intel Developer Catalog account and then search the catalog for “Intel® Smart Edge Open".

The current Reference Implementations are:

- [Wireless Network-Ready Intelligent Traffic Management](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/wireless-network-ready-intelligent-traffic-management.html): this RI is designed to detect and track vehicles and pedestrians. It provides the intelligence required to estimate a safety metric for an intersection. The goal is to use the collected intelligence to adjust traffic lights and optimize traffic flow at an intersection. 

- [Intelligent Connection Management xApp](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/intelligent-connection-management.html): this RI uses a deep reinforcement learning (DRL) algorithm with a graph neural network (GNN) model to implement the automatic handover of traffic between cells in a 5G wireless network. The goal is to improve connection quality and reduce latency.

- [Wireless Network Ready PCB Defect Detection](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/wireless-network-ready-pcb-defect-detection.html): this factory-floor RI detects two types of PCB defects during a manufacturing process – missing components and solder-bridge short circuits. The goal is to catch defects early to cut costs and improve product quality.

- [Smart VR - Live Streaming of Immersive Media](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/smart-vr-live-streaming-of-immersive-media.html): this RI ingests video data from multiple streams and uses the power of the edge platform to construct a 360-degree view and provide an interactive experience based on Field of View (FoV) request.

- [Telehealth Remote Monitoring](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/telehealth-remote-monitoring.html): this RI leverages the Intel® Collaboration Suite for WebRTC to set up and manage sessions between users that could include clinicians, patients, and caregivers. A variety of end-point devices like cameras, smartphones, and laptops are accommodated.

- [Network Optimization and AI Inferencing Management for Telepathology Reference Implementation](https://www.intel.com/content/www/us/en/developer/articles/reference-implementation/network-optimization-ai-inferencing-telepathology.html): this RI enables digital pathology through lab analysis automation. It showcases how an optimized cloud-native software architecture can simplify and automate networking challenges and optimize AI model deployment and management for digital pathology within a hospital system.  

All RIs have been tested to run on of the Intel® Smart Edge Developer Experience Kit, and a few of them have also been tested to run on other Experience Kits (EKs) as shown in the table below:

<div class="responsiveTable">
<table class="docTable">
    <thead>
        <tr>
            <th class="uk-text-center" style="min-width:258px">Reference<br>Implementation</th>
            <th class="uk-text-center" style="min-width:145px">Developer<br>Experience Kit</th>
            <th class="uk-text-center" style="min-width:155px">Private Wireless<br>Experience Kit</th>
            <th class="uk-text-center" style="min-width:260px">Secure Access Service Edge (SASE) Experience Kit</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Wireless Network-Ready Intelligent Traffic Management</td>
            <td class="uk-text-center">v 22.03.01</td>
            <td>v 22.04</td>
            <td></td>
        </tr>
        <tr>
            <td>Intelligent Connection Management xApp</td>
            <td class="uk-text-center">v 21.12.02</td>
            <td class="uk-text-center"></td>
            <td></td>
        </tr>
        <tr>
            <td>Wireless Network-Ready PCB Defect Detection</td>
            <td class="uk-text-center">v 22.03.01</td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>Smart VR – Live Streaming</td>
            <td class="uk-text-center">v 21.12.02</td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td>Telehealth Remote Monitoring</td>
            <td class="uk-text-center">v 21.12.02</td>
            <td></td>
            <td class="uk-text-center">v 22.05</td>
        </tr>
        <tr>
            <td>Network Optimization and AI Inferencing Management for Telepathology Reference Implementation</td>
            <td class="uk-text-center">v 21.03</td>
            <td></td>
            <td class="uk-text-center"></td>
        </tr>

    </tbody>
</table>
</div>
