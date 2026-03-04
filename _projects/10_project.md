---
layout: page
title: QGroundControl — EU Regulatory Extensions
img: assets/img/qgc_ui.png
importance: 2
category: fun
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            A fork of the open-source <strong>QGroundControl</strong> ground station software, extended
            to meet <strong>EU drone regulatory requirements</strong> and to expand the operational
            capabilities of the ground control interface for professional UAV deployments.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/qgroundcontrolUP" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
    </div>
</div>

---

## Overview

QGroundControl is a widely used open-source ground station application for MAVLink-based UAVs. This fork extends the original codebase with a set of features targeting compliance with **EU drone regulations** (EU 2019/947 and related provisions) and improved situational awareness for operators, without altering the core flight stack or mission planning logic.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/qgc_ui.png" title="Extended QGroundControl UI — second camera feed, geofence overlay, and NTRIP status panel" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

---

## Features Added

**NTRIP Client Integration.** An embedded NTRIP (Networked Transport of RTCM via Internet Protocol) client streams real-time GNSS correction data directly from a caster to the connected autopilot, enabling centimeter-level RTK positioning without requiring a separate correction device or external software.

**Second Camera Feed.** The UI was extended to display a second video stream alongside the primary camera, allowing simultaneous monitoring of a payload camera and a navigation camera — or any two independent video sources — within the same ground station window.

**Geofencing.** An interactive geofence editor was added to the map view, allowing operators to define both inclusive and exclusion zones that are enforced on the vehicle side. Zones are visually overlaid on the map and synchronized with the flight controller before takeoff.

**EU Regulatory Limits.** Hard and soft altitude warnings enforce the 120-meter AGL limit mandated for drone operations in the EU Open Category. The interface displays a continuous altitude indicator with color-coded thresholds and triggers a configurable warning when the limit is approached or exceeded. Additional regulatory hints cover horizontal distance constraints and visual line-of-sight reminders aligned with EU 2019/947 operational requirements.

**UI Improvements.** Several layout and usability changes were made to the main flight view, including reorganized telemetry panels, improved readability of critical flight data, and cleaner integration of the new widgets into the existing QML-based interface.