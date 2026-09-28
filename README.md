# FPV-Droneframe-CAD
+3.5-Inch FPV Drone openScad model designed by me made on openScad+


// Parametric small FPV drone frame (OpenSCAD)
// True-X, ~3" class. Flat bottom + top plates (print flat, or export the
// 2D shapes to DXF and cut from carbon fiber), standoffs, camera side plates.
//
// Defaults: 140 mm motor-to-motor diagonal, 12x12 motor pattern (M2),
// 20x20 flight-controller stack (M3), 19 mm micro camera.
// Check every hole pattern against your own parts before building!

/* [Part to show] */
part = "assembly"; // [assembly, bottom, top, camera_plate, bottom_2d, top_2d, camera_plate_2d]

/* [Frame size] */
wheelbase   = 140;   // motor to motor, diagonal
plate_t     = 3;     // plate thickness (3mm print, 3-4mm carbon)
arm_w       = 10;    // arm width
pad_d       = 20;    // motor pad diameter
body_l      = 56;    // center body length (front-back)
body_w      = 36;    // center body width
body_r      = 8;

/* [Stack / standoffs] */
stack_pattern = 20;    // 20 or 25.5 (mm between holes, square)
stack_hole_d  = 3.2;
standoff_x    = 22;    // standoff hole distance from center (front-back)
standoff_y    = 13;    // standoff hole distance from center (left-right)
standoff_hole = 3.2;
standoff_d    = 5.5;
standoff_h    = 26;    // gap between plates

/* [Motors] */
motor_pattern = 12;    // 12 = 1404/1507, 9 = 1103/1204, 16 = 22xx
motor_hole_d  = 2.3;
motor_center  = 7;     // center hole for the motor bell / shaft
prop_d        = 76;    // 3" props (only used for the preview)

/* [Camera] */
cam_width   = 19;      // 19 mm micro camera
cam_hole_d  = 2.3;
cam_hole_sp = 19;      // hole spacing between camera side plates hardware (vertical pivot hole)
cam_plate_r = 14;

/* [Battery strap] */
strap_w = 16;
strap_h = 2.6;
strap_gap = 14;        // distance of slots from center along the body

$fn = 64;

// ---------- Helpers ----------
motor_angles = [45, 135, 225, 315];
function motor_pos(a) = [wheelbase / 2 * cos(a), wheelbase / 2 * sin(a)];

module rrect(l, w, r) {
    offset(r) square([l - 2*r, w - 2*r], center = true);
}

module square_holes(s, d) {
    for (x = [-1, 1], y = [-1, 1])
        translate([x * s / 2, y * s / 2]) circle(d = d, $fn = 32);
}

module standoff_holes(d) {
    for (x = [-1, 1], y = [-1, 1])
        translate([x * standoff_x, y * standoff_y]) circle(d = d, $fn = 32);
}

// ---------- 2D shapes ----------
module bottom_2d() {
    difference() {
        union() {
            rrect(body_l, body_w, body_r);
            for (a = motor_angles)
                hull() {
                    circle(d = arm_w + 6);
                    translate(motor_pos(a)) circle(d = pad_d);
                }
        }
        // stack holes + wire opening
        square_holes(stack_pattern, stack_hole_d);
        standoff_holes(standoff_hole);
        // motor holes
        for (a = motor_angles)
            translate(motor_pos(a)) rotate(a) {
                square_holes(motor_pattern, motor_hole_d);
                circle(d = motor_center, $fn = 32);
            }
        // battery strap slots
        for (s = [-1, 1])
            translate([s * strap_gap, 0])
                square([strap_h, strap_w], center = true);
        // weight-saving holes in the arms
        for (a = motor_angles)
            hull() {
                translate(motor_pos(a) * 0.35) circle(d = 3.5, $fn = 24);
                translate(motor_pos(a) * 0.6) circle(d = 3.5, $fn = 24);
            }
    }
}

module top_2d() {
    difference() {
        rrect(body_l, body_w - 4, body_r);
        standoff_holes(standoff_hole);
        square_holes(stack_pattern, stack_hole_d);
        // antenna / wire holes
        translate([-body_l / 2 + 5, 0]) circle(d = 4, $fn = 24);
        // weight-saving cutout
        for (s = [-1, 1])
            translate([s * 13, 0]) rrect(8, 14, 2);
        // battery strap slots
        for (s = [-1, 1])
            translate([s * strap_gap, 0])
                square([strap_h, strap_w], center = true);
    }
}

module camera_plate_2d() {
    difference() {
        hull() {
            translate([0, 0]) circle(d = cam_plate_r * 2);
            translate([-22, -14]) circle(d = 8);
            translate([-22, 14]) circle(d = 8);
        }
        circle(d = cam_hole_d, $fn = 24);
        translate([-22, -14]) circle(d = standoff_hole - 0.4, $fn = 24);
        translate([-22, 14]) circle(d = standoff_hole - 0.4, $fn = 24);
    }
}

// ---------- 3D parts ----------
module bottom()       { linear_extrude(plate_t) bottom_2d(); }
module top()          { linear_extrude(plate_t) top_2d(); }
module camera_plate() { linear_extrude(plate_t) camera_plate_2d(); }

module standoffs() {
    for (x = [-1, 1], y = [-1, 1])
        translate([x * standoff_x, y * standoff_y, 0])
            difference() {
                cylinder(d = standoff_d, h = standoff_h);
                translate([0, 0, -1]) cylinder(d = standoff_hole - 0.6, h = standoff_h + 2, $fn = 24);
            }
}

// ---------- Preview extras ----------
module motors_and_props() {
    for (a = motor_angles)
        translate(motor_pos(a)) {
            color("silver") translate([0, 0, plate_t]) cylinder(d = 14, h = 8);
            color("red", 0.25) translate([0, 0, plate_t + 9]) cylinder(d = prop_d, h = 1);
        }
}

// ---------- Output ----------
if (part == "bottom") {
    bottom();
} else if (part == "top") {
    top();
} else if (part == "camera_plate") {
    camera_plate();
} else if (part == "bottom_2d") {
    bottom_2d();
} else if (part == "top_2d") {
    top_2d();
} else if (part == "camera_plate_2d") {
    camera_plate_2d();
} else {
    color("dimgray") bottom();
    color("gray") translate([0, 0, plate_t]) standoffs();
    color("dimgray") translate([0, 0, plate_t + standoff_h]) top();
    // camera side plates (vertical, at the front)
    for (s = [-1, 1])
        color("orange")
            translate([body_l / 2 + 4, s * (cam_width / 2 + plate_t / 2) - plate_t / 2 * 0, plate_t + standoff_h / 2])
                rotate([90, 0, 0]) translate([0, 0, -plate_t / 2]) linear_extrude(plate_t) camera_plate_2d();
    motors_and_props();
}
