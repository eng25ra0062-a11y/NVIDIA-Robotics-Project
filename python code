import rclpy
from rclpy.node import Node
from sensor_msgs.msg import JointState


class PandaPickPlace(Node):

    def __init__(self):
        super().__init__("panda_pick_place")

        self.pub = self.create_publisher(
            JointState,
            "/joint_command",
            10
        )

        self.joint_names = [
            "panda_joint1",
            "panda_joint2",
            "panda_joint3",
            "panda_joint4",
            "panda_joint5",
            "panda_joint6",
            "panda_joint7",
            "panda_finger_joint1",
            "panda_finger_joint2"
        ]

        self.gripper_open = 0.04
        self.gripper_closed = 0.0

        # HOME
        self.home = [
            0.0,
            -0.9987,
            0.0001,
            -2.2021,
            0.0033,
            2.3995,
            0.8
        ]

        # PICK
        self.pre_grasp = [
            0.72,
            0.72,
            0.00,
            -1.65,
            0.00,
            2.25,
            0.785
        ]
        
        self.grasp = [
            0.72,
            0.95,
            0.00,
            -1.42,
            0.00,
            2.30,
            0.785
        ]

        # LIFT — raise arm high after grasping for clean aesthetic
        self.lift = [
            0.72,
            0.10,   # arm raised high above table
            0.00,
            -2.20,  # elbow tucked tight
            0.00,
            2.00,
            0.785
        ]

        # PLACE PRE — swing to tray at elevated height
        self.place_pre = [
            -1.50,  # adjusted for tray center
            0.35,   # stay elevated during swing
            0.00,
            -1.50,  # elbow moderately extended for transit
            0.00,
            2.40,   # wrist keeps gripper down
            0.785
        ]
        
        # PLACE — descend and fully extend into center of tray
        self.place = [
            -1.50,  # same centered rotation
            1.15,   # lean far forward toward tray
            0.00,
            -0.80,  # elbow nearly straight — maximum extension
            0.00,
            2.50,   # wrist compensates to keep gripper pointing down
            0.785
        ]

    def send_pose(self, arm_pose, finger_width, steps=50):

        msg = JointState()

        msg.header.stamp = self.get_clock().now().to_msg()

        msg.name = self.joint_names

        msg.position = arm_pose + [
            finger_width,
            finger_width
        ]

        for _ in range(steps):
            self.pub.publish(msg)
            rclpy.spin_once(self, timeout_sec=0.02)

    def status(self, text, pause=0.5):

        print(f"[INFO] {text}...")

        import time
        time.sleep(pause)

    def run(self):

        self.status("MOVING TO HOME", 1.0)
        self.send_pose(self.home, self.gripper_open)

        self.status("APPROACHING OBJECT", 0.5)
        self.send_pose(self.pre_grasp, self.gripper_open)

        self.status("REACHING FOR GRASP", 0.5)
        self.send_pose(self.grasp, self.gripper_open)

        self.status("CLOSING GRIPPER", 0.8)
        self.send_pose(self.grasp, self.gripper_closed)

        self.status("LIFTING OBJECT", 0.5)
        self.send_pose(self.lift, self.gripper_closed, steps=80)

        self.status("TRANSPORTING TO TRAY", 0.3)
        self.send_pose(self.place_pre, self.gripper_closed, steps=90)

        self.status("LOWERING INTO TRAY", 0.3)
        self.send_pose(self.place, self.gripper_closed, steps=80)

        self.status("RELEASING OBJECT", 0.5)
        self.send_pose(self.place, self.gripper_open)

        self.status("RETURNING HOME", 0.5)
        self.send_pose(self.home, self.gripper_open)

        self.status("PICK AND PLACE COMPLETE!", 0.0)


def main():

    rclpy.init()

    node = PandaPickPlace()

    try:
        node.run()

    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == "__main__":
    main()
