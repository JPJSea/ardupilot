Changes in main since initial commit
```bash
$ git diff --name-only  96882ed0aeeb822dd80575312b782fe47c352b25 HEAD
ArduSub/Parameters.cpp
README.md
README_Ardupilot.md
Tools/environment_install/install-prereqs-ubuntu-py3.sh
libraries/AP_Motors/AP_Motors6DOF.cpp
libraries/AP_Motors/AP_Motors6DOF.h
test.md
waf-py3
```

Of these `test.md`, `waf-py3` and
`Tools/environment_install/install-prereqs-ubuntu-py3.sh` can probably be removed.
`README_Ardupilot.md` is a duplicate of original README and can go aswell.

These files will be copied into fork's main:
* `libraries/AP_Motors/AP_Motors6DOF.cpp`
* `libraries/AP_Motors/AP_Motors6DOF.h`
* `ArduSub/Parameters.cpp`
* `README.md`

Copying files into fork's main does not work there are too many changes in the
main branch that are overwritten.

Changing files manually:
```
$ git diff  96882ed0aeeb822dd80575312b782fe47c352b25 HEAD -- ArduSub/Parameters.cpp
diff --git a/ArduSub/Parameters.cpp b/ArduSub/Parameters.cpp
index 1f8dfa938..188331b71 100644
--- a/ArduSub/Parameters.cpp
+++ b/ArduSub/Parameters.cpp
@@ -286,8 +286,8 @@ const AP_Param::Info Sub::var_info[] = {
     // @Description: Set this parameter according to your vehicle/motor configuration
     // @User: Standard
     // @RebootRequired: True
-    // @Values: 0:BlueROV1, 1:Vectored, 2:Vectored_6DOF, 3:Vectored_6DOF_90, 4:SimpleROV-3, 5:SimpleROV-4, 6:SimpleROV-5, 7:Custom
-    GSCALAR(frame_configuration, "FRAME_CONFIG", AP_Motors6DOF::SUB_FRAME_VECTORED),
+    // @Values: 0:BlueROV1, 1:Vectored, 2:Vectored_6DOF, 3:Vectored_6DOF_90, 4:SimpleROV-3, 5:SimpleROV-4, 6:SimpleROV-5, 7:Custom, 8:JOYSTICK_PWM_CONTROL, 9:ROS_PWM_CONTROL
+    GSCALAR(frame_configuration, "FRAME_CONFIG", AP_Motors6DOF::SUB_FRAME_CUSTOM),

     // @Group: BTN0_
     // @Path: ../libraries/AP_JSButton/AP_JSButton.cpp
```

```
$ git diff  96882ed0aeeb822dd80575312b782fe47c352b25 HEAD -- libraries/AP_Motors/AP_Motors6DOF.cpp
diff --git a/libraries/AP_Motors/AP_Motors6DOF.cpp b/libraries/AP_Motors/AP_Motors6DOF.cpp
index f2588aa23..e099a34f8 100644
--- a/libraries/AP_Motors/AP_Motors6DOF.cpp
+++ b/libraries/AP_Motors/AP_Motors6DOF.cpp
@@ -171,9 +171,24 @@ void AP_Motors6DOF::setup_motors(motor_frame_class frame_class, motor_frame_type
         add_motor_raw_6dof(AP_MOTORS_MOT_6,     -1.0f,          0,              0,              -1.0f,              0,                  0,              6);
         break;

+    case SUB_FRAME_ROS_PWM_CONTROL:
+        //all motors are disabled.
+        break;
+
+    case SUB_FRAME_JOYSTICK_PWM_CONTROL:
+        add_motor_raw_6dof(AP_MOTORS_MOT_1,     0,              0,              0,              0,                   1.0f,             0,             1);
+        add_motor_raw_6dof(AP_MOTORS_MOT_2,     0,              0,              0,              0,                   0,                1.0f,          2);
+        add_motor_raw_6dof(AP_MOTORS_MOT_3,     0,              0,              1.0f,           0,                   0,                0,             3);
+        add_motor_raw_6dof(AP_MOTORS_MOT_4,     0,              0,              0,              1.0f,                0,                0,             4);
+        break;
+
     case SUB_FRAME_CUSTOM:
-        // Put your custom motor setup here
-        //break;
+        add_motor_raw_6dof(AP_MOTORS_MOT_1,     0,              0,              -1.0f,           0,                  -1.0f,               1.0f,           1);
+        add_motor_raw_6dof(AP_MOTORS_MOT_2,     0,              0,              1.0f,           0,                   -1.0f,               -1.0f,           2);
+        add_motor_raw_6dof(AP_MOTORS_MOT_3,     0,              0,              1.0f,           0,                  1.0f,               1.0f,           3);
+        add_motor_raw_6dof(AP_MOTORS_MOT_4,     0,              0,              -1.0f,           0,                  1.0f,               -1.0f,           4);
+        break;
+

     case SUB_FRAME_SIMPLEROV_3:
         add_motor_raw_6dof(AP_MOTORS_MOT_1,     0,              0,              -1.0f,          0,                  1.0f,               0,              1);
```

```
$ git diff  96882ed0aeeb822dd80575312b782fe47c352b25 HEAD -- libraries/AP_Motors/AP_Motors6DOF.h
diff --git a/libraries/AP_Motors/AP_Motors6DOF.h b/libraries/AP_Motors/AP_Motors6DOF.h
index ca9d6abc1..1217d35ef 100644
--- a/libraries/AP_Motors/AP_Motors6DOF.h
+++ b/libraries/AP_Motors/AP_Motors6DOF.h
@@ -26,7 +26,9 @@ public:
         SUB_FRAME_SIMPLEROV_3,
         SUB_FRAME_SIMPLEROV_4,
         SUB_FRAME_SIMPLEROV_5,
-        SUB_FRAME_CUSTOM
+        SUB_FRAME_CUSTOM,
+        SUB_FRAME_JOYSTICK_PWM_CONTROL,
+        SUB_FRAME_ROS_PWM_CONTROL
     } sub_frame_t;

     // Override parent
```
