from machine import Pin, PWM
import time
import math

class Motor:
    def __init__(self, en_pin, in1_pin, in2_pin, pwm_freq=1000):
        self.en = PWM(Pin(en_pin))
        self.en.freq(pwm_freq)
        self.in1 = Pin(in1_pin, Pin.OUT)
        self.in2 = Pin(in2_pin, Pin.OUT)
        self.stop()

    def forward(self, speed=5000):
        self.in1.value(0)
        self.in2.value(1)
        self.set_speed(speed)

    def backward(self, speed=5000):
        self.in1.value(1)
        self.in2.value(0)
        self.set_speed(speed)

    def stop(self):
        self.set_speed(0)
        self.in1.value(0)
        self.in2.value(0)

    def set_speed(self, speed):
        if speed < 0:
            speed = 0
        elif speed > 65535:
            speed = 65535
        self.en.duty_u16(speed)

motor0 = Motor(2, 0, 1)
motor1 = Motor(8, 6, 7)
motor2 = Motor(12, 10, 11)

def stop_all():
    motor0.stop()
    motor1.stop()
    motor2.stop()

def normalize_speeds(speeds, max_val=65535):
    max_speed = max(abs(s) for s in speeds)
    if max_speed > max_val:
        scale = max_val / max_speed
        return [int(s * scale) for s in speeds]
    else:
        return [int(s) for s in speeds]

def holonomic(direction_deg, speed):
    rad = math.radians(direction_deg)
    angle0 = rad
    angle1 = rad - 2 * math.pi / 3
    angle2 = rad - 4 * math.pi / 3

    v0 = speed * math.cos(angle0)
    v1 = speed * math.cos(angle1)
    v2 = speed * math.cos(angle2)

    v0, v1, v2 = normalize_speeds([v0, v1, v2])

    if v0 >= 0:
        motor0.forward(v0)
    else:
        motor0.backward(abs(v0))

    if v1 >= 0:
        motor1.forward(v1)
    else:
        motor1.backward(abs(v1))

    if v2 >= 0:
        motor2.forward(v2)
    else:
        motor2.backward(abs(v2))

def motor_ctrl(motor, speed):
    if speed >= 0:
        motor.forward(min(speed, 65535))
    else:
        motor.backward(min(abs(speed), 65535))

def non_holonomic(forward_speed, lateral_speed, angular_speed):
    motor0_speed = forward_speed + angular_speed
    motor1_speed = lateral_speed + angular_speed
    motor2_speed = -forward_speed + angular_speed

    motor0_speed, motor1_speed, motor2_speed = normalize_speeds([motor0_speed, motor1_speed, motor2_speed])

    motor_ctrl(motor0, motor0_speed)
    motor_ctrl(motor1, motor1_speed)
    motor_ctrl(motor2, motor2_speed)

# 테스트 동작
holonomic(0, 40000)   # 0도 방향 60% 정도 속도 (65535 * 0.6 ≈ 39321)
time.sleep(2)
holonomic(270, 40000) # 270도 방향 60% 속도
time.sleep(2)
non_holonomic(0, 0, 20000)  # 제자리 회전
time.sleep(2)
stop_all()

