
# 第二部分：完整代码实现

本部分提供机器狗核心模块的**完整可运行代码**，覆盖驱动层、算法层、应用层三层结构，可直接复制参考并在 STM32 HAL 库环境下使用。

---

## 11. 核心模块完整代码

### 11.1 PCA9685 舵机驱动完整代码

**`pca9685.h`**

```c
#ifndef __PCA9685_H
#define __PCA9685_H

#include "stm32f4xx_hal.h"

/* PCA9685 I2C 地址（7位，HAL 库需要 << 1） */
#define PCA9685_ADDR            (0x40 << 1)

/* 关键寄存器地址 */
#define PCA9685_MODE1           0x00
#define PCA9685_MODE2           0x01
#define PCA9685_PRESCALE        0xFE
#define PCA9685_LED0_ON_L       0x06
#define PCA9685_LED0_ON_H       0x07
#define PCA9685_LED0_OFF_L      0x08
#define PCA9685_LED0_OFF_H      0x09

/* 内部振荡频率 */
#define PCA9685_OSC_FREQ        25000000UL

/* 通道数量 */
#define PCA9685_CHANNELS        16

/* 舵机角度范围对应 PWM 计数（基于 50Hz） */
#define SERVO_MIN_PULSE         102     /* 0.5ms */
#define SERVO_MAX_PULSE         512     /* 2.5ms */
#define SERVO_ANGLE_MAX         180.0f

/* API */
HAL_StatusTypeDef PCA9685_Init(I2C_HandleTypeDef *hi2c, float freq_hz);
HAL_StatusTypeDef PCA9685_SetFreq(I2C_HandleTypeDef *hi2c, float freq_hz);
HAL_StatusTypeDef PCA9685_SetPWM(I2C_HandleTypeDef *hi2c, uint8_t ch, uint16_t on, uint16_t off);
HAL_StatusTypeDef PCA9685_SetAngle(I2C_HandleTypeDef *hi2c, uint8_t ch, float angle);
HAL_StatusTypeDef PCA9685_SetAllAngle(I2C_HandleTypeDef *hi2c, float angle);
HAL_StatusTypeDef PCA9685_Sleep(I2C_HandleTypeDef *hi2c);
HAL_StatusTypeDef PCA9685_WakeUp(I2C_HandleTypeDef *hi2c);

#endif
```

**`pca9685.c`**

```c
#include "pca9685.h"
#include <math.h>

static HAL_StatusTypeDef PCA9685_WriteReg(I2C_HandleTypeDef *hi2c, uint8_t reg, uint8_t val) {
    return HAL_I2C_Mem_Write(hi2c, PCA9685_ADDR, reg, I2C_MEMADD_SIZE_8BIT, &val, 1, 100);
}

static HAL_StatusTypeDef PCA9685_ReadReg(I2C_HandleTypeDef *hi2c, uint8_t reg, uint8_t *val) {
    return HAL_I2C_Mem_Read(hi2c, PCA9685_ADDR, reg, I2C_MEMADD_SIZE_8BIT, val, 1, 100);
}

HAL_StatusTypeDef PCA9685_SetFreq(I2C_HandleTypeDef *hi2c, float freq_hz) {
    if (freq_hz < 24) freq_hz = 24;
    if (freq_hz > 1526) freq_hz = 1526;

    /* prescale = round(osc / (4096 * freq)) - 1 */
    float prescaleval = (PCA9685_OSC_FREQ / (4096.0f * freq_hz)) - 1.0f;
    uint8_t prescale = (uint8_t)(prescaleval + 0.5f);

    uint8_t oldmode;
    if (PCA9685_ReadReg(hi2c, PCA9685_MODE1, &oldmode) != HAL_OK) return HAL_ERROR;

    uint8_t newmode = (oldmode & 0x7F) | 0x10;           /* sleep 位置1，预分频才可写 */
    PCA9685_WriteReg(hi2c, PCA9685_MODE1, newmode);
    PCA9685_WriteReg(hi2c, PCA9685_PRESCALE, prescale);
    PCA9685_WriteReg(hi2c, PCA9685_MODE1, oldmode);
    HAL_Delay(5);
    PCA9685_WriteReg(hi2c, PCA9685_MODE1, oldmode | 0xA1); /* 自动增量 + 重启 */
    return HAL_OK;
}

HAL_StatusTypeDef PCA9685_Init(I2C_HandleTypeDef *hi2c, float freq_hz) {
    if (PCA9685_WriteReg(hi2c, PCA9685_MODE1, 0x00) != HAL_OK) return HAL_ERROR;
    return PCA9685_SetFreq(hi2c, freq_hz);
}

HAL_StatusTypeDef PCA9685_SetPWM(I2C_HandleTypeDef *hi2c, uint8_t ch, uint16_t on, uint16_t off) {
    uint8_t buf[4] = { on & 0xFF, (on >> 8) & 0x0F, off & 0xFF, (off >> 8) & 0x0F };
    uint8_t reg = PCA9685_LED0_ON_L + 4 * ch;
    return HAL_I2C_Mem_Write(hi2c, PCA9685_ADDR, reg, I2C_MEMADD_SIZE_8BIT, buf, 4, 100);
}

HAL_StatusTypeDef PCA9685_SetAngle(I2C_HandleTypeDef *hi2c, uint8_t ch, float angle) {
    if (angle < 0) angle = 0;
    if (angle > SERVO_ANGLE_MAX) angle = SERVO_ANGLE_MAX;
    uint16_t off = (uint16_t)(SERVO_MIN_PULSE + (SERVO_MAX_PULSE - SERVO_MIN_PULSE) * angle / SERVO_ANGLE_MAX);
    return PCA9685_SetPWM(hi2c, ch, 0, off);
}

HAL_StatusTypeDef PCA9685_SetAllAngle(I2C_HandleTypeDef *hi2c, float angle) {
    for (uint8_t i = 0; i < PCA9685_CHANNELS; i++) {
        if (PCA9685_SetAngle(hi2c, i, angle) != HAL_OK) return HAL_ERROR;
    }
    return HAL_OK;
}
```

**要点解读**：
- `PCA9685_SetFreq` 写预分频器前必须先把芯片置 sleep，否则寄存器只读。
- 地址宏 `<< 1` 是 HAL 库的规则（最低位给读/写位）。
- 用 `HAL_I2C_Mem_Write` 一次写 4 字节比 4 次单写高效（依赖 PCA9685 的自动增量 AI=1）。

---

### 11.2 MPU6050 驱动完整代码

**`mpu6050.h`**

```c
#ifndef __MPU6050_H
#define __MPU6050_H

#include "stm32f4xx_hal.h"

#define MPU6050_ADDR            (0x68 << 1)

#define MPU6050_SMPLRT_DIV      0x19
#define MPU6050_CONFIG          0x1A
#define MPU6050_GYRO_CONFIG     0x1B
#define MPU6050_ACCEL_CONFIG    0x1C
#define MPU6050_ACCEL_XOUT_H    0x3B
#define MPU6050_GYRO_XOUT_H     0x43
#define MPU6050_PWR_MGMT_1      0x6B
#define MPU6050_WHO_AM_I        0x75

typedef struct {
    float ax, ay, az;   /* g */
    float gx, gy, gz;   /* deg/s */
    float temp;         /* C */
} MPU6050_Data_t;

HAL_StatusTypeDef MPU6050_Init(I2C_HandleTypeDef *hi2c);
HAL_StatusTypeDef MPU6050_ReadAll(I2C_HandleTypeDef *hi2c, MPU6050_Data_t *data);
uint8_t MPU6050_Check(I2C_HandleTypeDef *hi2c);

#endif
```

**`mpu6050.c`**

```c
#include "mpu6050.h"

#define ACCEL_SENS_2G   16384.0f
#define GYRO_SENS_250   131.0f

HAL_StatusTypeDef MPU6050_Init(I2C_HandleTypeDef *hi2c) {
    uint8_t v;
    if (HAL_I2C_Mem_Read(hi2c, MPU6050_ADDR, MPU6050_WHO_AM_I, 1, &v, 1, 100) != HAL_OK) return HAL_ERROR;
    if (v != 0x68) return HAL_ERROR;

    uint8_t config[][2] = {
        {MPU6050_PWR_MGMT_1, 0x00},
        {MPU6050_SMPLRT_DIV, 0x07},
        {MPU6050_CONFIG,     0x06},
        {MPU6050_GYRO_CONFIG, 0x00},
        {MPU6050_ACCEL_CONFIG, 0x00},
    };
    for (int i = 0; i < 5; i++) {
        if (HAL_I2C_Mem_Write(hi2c, MPU6050_ADDR, config[i][0], 1, &config[i][1], 1, 100) != HAL_OK)
            return HAL_ERROR;
    }
    return HAL_OK;
}

HAL_StatusTypeDef MPU6050_ReadAll(I2C_HandleTypeDef *hi2c, MPU6050_Data_t *data) {
    uint8_t buf[14];
    if (HAL_I2C_Mem_Read(hi2c, MPU6050_ADDR, MPU6050_ACCEL_XOUT_H, 1, buf, 14, 100) != HAL_OK)
        return HAL_ERROR;

    int16_t ax = (buf[0] << 8) | buf[1];
    int16_t ay = (buf[2] << 8) | buf[3];
    int16_t az = (buf[4] << 8) | buf[5];
    int16_t tr = (buf[6] << 8) | buf[7];
    int16_t gx = (buf[8] << 8) | buf[9];
    int16_t gy = (buf[10] << 8) | buf[11];
    int16_t gz = (buf[12] << 8) | buf[13];

    data->ax = ax / ACCEL_SENS_2G;
    data->ay = ay / ACCEL_SENS_2G;
    data->az = az / ACCEL_SENS_2G;
    data->gx = gx / GYRO_SENS_250;
    data->gy = gy / GYRO_SENS_250;
    data->gz = gz / GYRO_SENS_250;
    data->temp = tr / 340.0f + 36.53f;
    return HAL_OK;
}

uint8_t MPU6050_Check(I2C_HandleTypeDef *hi2c) {
    uint8_t v = 0;
    HAL_I2C_Mem_Read(hi2c, MPU6050_ADDR, MPU6050_WHO_AM_I, 1, &v, 1, 100);
    return v;
}
```

---

### 11.3 运动学逆解完整代码

**`kinematics.h`**

```c
#ifndef __KINEMATICS_H
#define __KINEMATICS_H

#define LEG_L1   50.0f
#define LEG_L2   60.0f

int InverseResolve(float x, float y, float *a1, float *a2);

#endif
```

**`kinematics.c`**

```c
#include "kinematics.h"
#include <math.h>

#define PI 3.14159265358979f
#define RAD2DEG(x)  ((x) * 180.0f / PI)

int InverseResolve(float x, float y, float *a1, float *a2) {
    float d2 = x * x + y * y;
    float d  = sqrtf(d2);

    float max_reach = LEG_L1 + LEG_L2;
    float min_reach = fabsf(LEG_L1 - LEG_L2);
    if (d > max_reach - 0.1f || d < min_reach + 0.1f) {
        return -1;
    }

    float cos_knee = (LEG_L1 * LEG_L1 + LEG_L2 * LEG_L2 - d2) / (2.0f * LEG_L1 * LEG_L2);
    if (cos_knee > 1.0f)  cos_knee = 1.0f;
    if (cos_knee < -1.0f) cos_knee = -1.0f;
    float knee = acosf(cos_knee);
    *a2 = RAD2DEG(PI - knee);

    float alpha = atan2f(x, y);
    float cos_beta = (LEG_L1 * LEG_L1 + d2 - LEG_L2 * LEG_L2) / (2.0f * LEG_L1 * d);
    if (cos_beta > 1.0f)  cos_beta = 1.0f;
    if (cos_beta < -1.0f) cos_beta = -1.0f;
    float beta = acosf(cos_beta);
    *a1 = RAD2DEG(alpha - beta) + 90.0f;

    if (*a1 < 0 || *a1 > 180 || *a2 < 0 || *a2 > 180) return -1;
    return 0;
}
```

---

### 11.4 舵机平滑过渡（带状态保存）

```c
#define NUM_SERVOS  16

static float g_current_angle[NUM_SERVOS] = {0};
static const float g_step[NUM_SERVOS]    = {
    1.0f, 1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 1.0f, 1.0f
};

int AngleMoveStep(I2C_HandleTypeDef *hi2c, uint8_t ch, float target) {
    float current = g_current_angle[ch];
    float diff = target - current;
    if (fabsf(diff) <= g_step[ch]) {
        PCA9685_SetAngle(hi2c, ch, target);
        g_current_angle[ch] = target;
        return 1;
    }
    current += (diff > 0) ? g_step[ch] : -g_step[ch];
    PCA9685_SetAngle(hi2c, ch, current);
    g_current_angle[ch] = current;
    return 0;
}

void AngleMoveAll(I2C_HandleTypeDef *hi2c, float targets[NUM_SERVOS], uint32_t tick_ms) {
    while (1) {
        int all_done = 1;
        for (int i = 0; i < NUM_SERVOS; i++) {
            if (!AngleMoveStep(hi2c, i, targets[i])) all_done = 0;
        }
        if (all_done) break;
        HAL_Delay(tick_ms);
    }
}
```

---

### 11.5 动作控制（以 DogRun 为例）

```c
typedef struct {
    float x_fl, y_fl;
    float x_fr, y_fr;
    float x_bl, y_bl;
    float x_br, y_br;
    uint16_t delay_ms;
} ActionFrame_t;

static const ActionFrame_t Trot_Forward[] = {
    { 20, 60, -20, 60,  -20, 60,  20, 60, 100 },
    { 30, 50, -20, 60,  -20, 60,  30, 50,  80 },
    { 30, 60, -20, 60,  -20, 60,  30, 60,  80 },
    {-20, 60,  30, 50,   30, 50, -20, 60,  80 },
    {-20, 60,  30, 60,   30, 60, -20, 60,  80 },
    { 0 }
};

static const uint8_t LEG_SERVO_MAP[4][2] = {
    {0, 1}, {2, 3}, {4, 5}, {6, 7}
};

static const float PX[8] = {0, 0, 0, 0, 0, 0, 0, 0};

void SetLegAngle(I2C_HandleTypeDef *hi2c, uint8_t leg, float a1, float a2) {
    uint8_t c1 = LEG_SERVO_MAP[leg][0];
    uint8_t c2 = LEG_SERVO_MAP[leg][1];
    PCA9685_SetAngle(hi2c, c1, a1 + PX[c1]);
    PCA9685_SetAngle(hi2c, c2, a2 + PX[c2]);
}

int ExecFrame(I2C_HandleTypeDef *hi2c, const ActionFrame_t *f) {
    float a1, a2;
    if (InverseResolve(f->x_fl, f->y_fl, &a1, &a2) < 0) return -1;
    SetLegAngle(hi2c, 0, a1, a2);
    if (InverseResolve(f->x_fr, f->y_fr, &a1, &a2) < 0) return -1;
    SetLegAngle(hi2c, 1, a1, a2);
    if (InverseResolve(f->x_bl, f->y_bl, &a1, &a2) < 0) return -1;
    SetLegAngle(hi2c, 2, a1, a2);
    if (InverseResolve(f->x_br, f->y_br, &a1, &a2) < 0) return -1;
    SetLegAngle(hi2c, 3, a1, a2);
    HAL_Delay(f->delay_ms);
    return 0;
}

void DogRun(I2C_HandleTypeDef *hi2c) {
    for (int i = 0; Trot_Forward[i].delay_ms != 0; i++) {
        ExecFrame(hi2c, &Trot_Forward[i]);
    }
}
```

---

### 11.6 UART 不定长接收（IDLE 中断 + DMA）

```c
#define UART_RX_BUF_SIZE   64

typedef struct {
    uint8_t  buf[UART_RX_BUF_SIZE];
    uint16_t len;
    volatile uint8_t ready;
} UartCmd_t;

UartCmd_t g_bt_cmd;
UartCmd_t g_voice_cmd;

void UartCmd_Init(UART_HandleTypeDef *huart, UartCmd_t *cmd) {
    memset(cmd, 0, sizeof(*cmd));
    __HAL_UART_ENABLE_IT(huart, UART_IT_IDLE);
    HAL_UARTEx_ReceiveToIdle_DMA(huart, cmd->buf, UART_RX_BUF_SIZE);
    __HAL_DMA_DISABLE_IT(huart->hdmarx, DMA_IT_HT);
}

void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size) {
    if (huart->Instance == USART2) {
        g_bt_cmd.len   = Size;
        g_bt_cmd.ready = 1;
        HAL_UARTEx_ReceiveToIdle_DMA(huart, g_bt_cmd.buf, UART_RX_BUF_SIZE);
        __HAL_DMA_DISABLE_IT(huart->hdmarx, DMA_IT_HT);
    } else if (huart->Instance == UART4) {
        g_voice_cmd.len   = Size;
        g_voice_cmd.ready = 1;
        HAL_UARTEx_ReceiveToIdle_DMA(huart, g_voice_cmd.buf, UART_RX_BUF_SIZE);
        __HAL_DMA_DISABLE_IT(huart->hdmarx, DMA_IT_HT);
    }
}
```

---

### 11.7 指令解析与调度（命令表模式）

```c
typedef void (*ActionFunc_t)(I2C_HandleTypeDef *);

typedef struct {
    const char  *keyword;
    ActionFunc_t func;
} CmdEntry_t;

void DogStand(I2C_HandleTypeDef *hi2c);
void DogRun  (I2C_HandleTypeDef *hi2c);
void DogBack (I2C_HandleTypeDef *hi2c);
void DogLeft (I2C_HandleTypeDef *hi2c);
void DogRight(I2C_HandleTypeDef *hi2c);
void DogSquat(I2C_HandleTypeDef *hi2c);
void DogWave (I2C_HandleTypeDef *hi2c);

static const CmdEntry_t g_cmd_table[] = {
    {"STAND",   DogStand},
    {"FORWARD", DogRun},
    {"BACK",    DogBack},
    {"LEFT",    DogLeft},
    {"RIGHT",   DogRight},
    {"SQUAT",   DogSquat},
    {"WAVE",    DogWave},
    {NULL, NULL}
};

void ParseAndExec(I2C_HandleTypeDef *hi2c, const char *str) {
    for (int i = 0; g_cmd_table[i].keyword != NULL; i++) {
        if (strstr(str, g_cmd_table[i].keyword)) {
            g_cmd_table[i].func(hi2c);
            printf("[CMD] %s executed\r\n", g_cmd_table[i].keyword);
            return;
        }
    }
    printf("[CMD] unknown: %s\r\n", str);
}
```

**主循环使用**：

```c
while (1) {
    if (g_bt_cmd.ready) {
        ParseAndExec(&hi2c1, (char*)g_bt_cmd.buf);
        memset(g_bt_cmd.buf, 0, UART_RX_BUF_SIZE);
        g_bt_cmd.ready = 0;
    }
    if (g_voice_cmd.ready) {
        ParseAndExec(&hi2c1, (char*)g_voice_cmd.buf);
        memset(g_voice_cmd.buf, 0, UART_RX_BUF_SIZE);
        g_voice_cmd.ready = 0;
    }
}
```

---

### 11.8 WS2812 SPI+DMA 稳定版

```c
#define LED_COUNT   5
#define BIT_PER_LED 24
#define WS_CODE_0   0x80
#define WS_CODE_1   0xF8

static uint8_t g_tx_buf[LED_COUNT * BIT_PER_LED + 40];
extern SPI_HandleTypeDef hspi1;

static void EncodeByte(uint8_t *dst, uint8_t val) {
    for (int i = 7; i >= 0; i--) {
        dst[7 - i] = (val & (1 << i)) ? WS_CODE_1 : WS_CODE_0;
    }
}

void WS2812_SetColor(uint8_t idx, uint8_t r, uint8_t g, uint8_t b) {
    if (idx >= LED_COUNT) return;
    uint8_t *p = &g_tx_buf[idx * BIT_PER_LED];
    EncodeByte(p,      g);
    EncodeByte(p + 8,  r);
    EncodeByte(p + 16, b);
}

void WS2812_Flush(void) {
    HAL_SPI_Transmit_DMA(&hspi1, g_tx_buf, sizeof(g_tx_buf));
}

void WS2812_Init(void) {
    memset(g_tx_buf, 0, sizeof(g_tx_buf));
}
```

---

# 第三部分：衍生功能 · 丰富项目亮点

挑 2~3 个实现，简历直接多出一页硬核亮点。

---

## 12. 姿态闭环（PID + MPU6050 互补滤波）

### 12.1 原理

舵机角度是开环的，推一下就摔。接入 MPU6050 估算 Pitch / Roll，通过 PID 实时调整腿高，实现"被推不倒"。

### 12.2 互补滤波估算欧拉角

```c
typedef struct {
    float pitch, roll;
    float dt;
    float alpha;
    uint32_t last_tick;
} AttitudeFilter_t;

void Attitude_Init(AttitudeFilter_t *af) {
    af->pitch = af->roll = 0;
    af->alpha = 0.98f;
    af->last_tick = HAL_GetTick();
}

void Attitude_Update(AttitudeFilter_t *af, MPU6050_Data_t *d) {
    uint32_t now = HAL_GetTick();
    af->dt = (now - af->last_tick) / 1000.0f;
    af->last_tick = now;

    float acc_roll  = atan2f(d->ay, d->az) * 57.2958f;
    float acc_pitch = atan2f(-d->ax, sqrtf(d->ay*d->ay + d->az*d->az)) * 57.2958f;

    float gyro_roll  = af->roll  + d->gx * af->dt;
    float gyro_pitch = af->pitch + d->gy * af->dt;

    af->roll  = af->alpha * gyro_roll  + (1 - af->alpha) * acc_roll;
    af->pitch = af->alpha * gyro_pitch + (1 - af->alpha) * acc_pitch;
}
```

### 12.3 增量式 PID

```c
typedef struct {
    float kp, ki, kd;
    float err_prev, err_prev2;
    float out_prev;
    float out_max, out_min;
} PID_t;

float PID_Inc(PID_t *p, float err) {
    float du = p->kp * (err - p->err_prev)
             + p->ki * err
             + p->kd * (err - 2*p->err_prev + p->err_prev2);
    p->err_prev2 = p->err_prev;
    p->err_prev  = err;
    p->out_prev += du;
    if (p->out_prev > p->out_max) p->out_prev = p->out_max;
    if (p->out_prev < p->out_min) p->out_prev = p->out_min;
    return p->out_prev;
}
```

### 12.4 姿态自稳循环

```c
AttitudeFilter_t g_att;
PID_t g_pid_pitch = {0.5f, 0.02f, 0.1f, 0, 0, 0, +20, -20};
PID_t g_pid_roll  = {0.5f, 0.02f, 0.1f, 0, 0, 0, +20, -20};

void StabilizeTask(void) {
    MPU6050_Data_t d;
    MPU6050_ReadAll(&hi2c1, &d);
    Attitude_Update(&g_att, &d);

    float dh_pitch = PID_Inc(&g_pid_pitch, 0 - g_att.pitch);
    float dh_roll  = PID_Inc(&g_pid_roll,  0 - g_att.roll);

    float h = 60.0f;
    float legs_y[4] = {
        h + dh_pitch - dh_roll,
        h + dh_pitch + dh_roll,
        h - dh_pitch - dh_roll,
        h - dh_pitch + dh_roll
    };
    for (int i = 0; i < 4; i++) {
        float a1, a2;
        if (InverseResolve(0, legs_y[i], &a1, &a2) == 0) {
            SetLegAngle(&hi2c1, i, a1, a2);
        }
    }
}
```

**调用频率**：10~50Hz。
**简历话术**：基于 MPU6050 + 互补滤波 + 增量式 PID 实现机身姿态闭环自稳，机器狗被外力推动后可自动恢复水平。

---

## 13. FreeRTOS 多任务重构

### 13.1 任务划分

| 任务 | 优先级 | 周期 | 职责 |
|------|--------|------|------|
| `StabilizeTask` | 3（高） | 20ms | 姿态闭环 |
| `ActionTask` | 2 | 事件 | 执行动作序列 |
| `UartRxTask` | 2 | 事件 | 解析串口指令 |
| `LedTask` | 1 | 50ms | 状态灯刷新 |
| `LogTask` | 1 | 100ms | 调试日志 |

### 13.2 关键代码

```c
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "semphr.h"

QueueHandle_t xCmdQueue;
SemaphoreHandle_t xI2CMutex;

void vStabilizeTask(void *arg) {
    const TickType_t period = pdMS_TO_TICKS(20);
    TickType_t last = xTaskGetTickCount();
    for (;;) {
        xSemaphoreTake(xI2CMutex, portMAX_DELAY);
        StabilizeTask();
        xSemaphoreGive(xI2CMutex);
        vTaskDelayUntil(&last, period);
    }
}

void vActionTask(void *arg) {
    uint8_t cmd;
    for (;;) {
        if (xQueueReceive(xCmdQueue, &cmd, portMAX_DELAY) == pdTRUE) {
            xSemaphoreTake(xI2CMutex, portMAX_DELAY);
            switch (cmd) {
                case 1: DogRun(&hi2c1);   break;
                case 2: DogBack(&hi2c1);  break;
                case 3: DogLeft(&hi2c1);  break;
                case 4: DogRight(&hi2c1); break;
            }
            xSemaphoreGive(xI2CMutex);
        }
    }
}

int main(void) {
    /* HAL init ... */
    xCmdQueue = xQueueCreate(8, sizeof(uint8_t));
    xI2CMutex = xSemaphoreCreateMutex();
    xTaskCreate(vStabilizeTask, "STAB", 512, NULL, 3, NULL);
    xTaskCreate(vActionTask,    "ACT",  512, NULL, 2, NULL);
    xTaskCreate(vUartRxTask,    "UART", 256, NULL, 2, NULL);
    xTaskCreate(vLedTask,       "LED",  128, NULL, 1, NULL);
    vTaskStartScheduler();
    while (1);
}
```

**简历话术**：基于 FreeRTOS 将系统重构为多任务并行架构，使用互斥量保护 I2C 总线，队列解耦通信与执行，系统响应延迟低于 20ms。

---

## 14. MQTT 云端控制

### 14.1 Topic 设计

| Topic | 方向 | Payload |
|-------|------|---------|
| `dog/{id}/cmd` | 下行 | `{"cmd":"FORWARD"}` |
| `dog/{id}/state` | 上行 | `{"pitch":1.2,"roll":-0.3,"battery":85}` |
| `dog/{id}/event` | 上行 | `{"event":"fall","ts":1700000000}` |

### 14.2 ESP32 核心代码

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

WiFiClient espClient;
PubSubClient client(espClient);

void mqttCallback(char *topic, byte *payload, unsigned int length) {
    String msg = "";
    for (int i = 0; i < length; i++) msg += (char)payload[i];
    int p = msg.indexOf("cmd");
    if (p < 0) return;
    int q1 = msg.indexOf('"', p + 5);
    int q2 = msg.indexOf('"', q1 + 1);
    String cmd = msg.substring(q1 + 1, q2);
    Serial2.print(cmd + "\n");
}

void setup() {
    Serial.begin(115200);
    Serial2.begin(115200, SERIAL_8N1, 16, 17);
    WiFi.begin("YOUR_SSID", "YOUR_PASS");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    client.setServer("broker.emqx.io", 1883);
    client.setCallback(mqttCallback);
}

void loop() {
    if (!client.connected()) {
        if (client.connect("dog-0001")) {
            client.subscribe("dog/0001/cmd");
        }
    }
    client.loop();

    static unsigned long last = 0;
    if (millis() - last > 1000) {
        if (Serial2.available()) {
            String state = Serial2.readStringUntil('\n');
            client.publish("dog/0001/state", state.c_str());
        }
        last = millis();
    }
}
```

**简历话术**：基于 MQTT 协议实现手机 APP 与机器狗的端到端远程通信，支持云端指令下发与状态实时上报。

---

## 15. OTA 无线固件升级（ESP32）

```cpp
#include <ArduinoOTA.h>

void setupOTA() {
    ArduinoOTA.setHostname("dog-esp32");
    ArduinoOTA.setPassword("your_pass");
    ArduinoOTA.onStart([]() { Serial.println("OTA Start"); });
    ArduinoOTA.onEnd  ([]() { Serial.println("OTA End");   });
    ArduinoOTA.onProgress([](unsigned int p, unsigned int t) {
        Serial.printf("OTA %u%%\r\n", p * 100 / t);
    });
    ArduinoOTA.begin();
}

void loop() {
    ArduinoOTA.handle();
    /* ... */
}
```

**简历话术**：集成 Wi-Fi OTA，无需数据线即可远程刷写固件，提升运维效率。

---

## 16. S 曲线步态插值

```c
/* 三次 Hermite 平滑：t ∈ [0,1] 映射到 [0,1] 但端点导数为 0 */
float SCurve(float t) {
    return t * t * (3.0f - 2.0f * t);
}

void AngleMoveSCurve(I2C_HandleTypeDef *hi2c, uint8_t ch,
                     float target, int steps, uint16_t dt_ms) {
    float start = g_current_angle[ch];
    float span  = target - start;
    for (int i = 1; i <= steps; i++) {
        float t = (float)i / steps;
        float progress = SCurve(t);
        float ang = start + span * progress;
        PCA9685_SetAngle(hi2c, ch, ang);
        HAL_Delay(dt_ms);
    }
    g_current_angle[ch] = target;
}
```

**简历话术**：用三次 Hermite 平滑曲线替代线性插值，加速度连续，动作更接近生物运动。

---

## 17. OLED 本地仪表盘

SSD1306（I2C 0x3C）接入，显示姿态、电量、当前动作、联网状态。

```c
void OLED_UpdateDashboard(float pitch, float roll, const char* action, int batt) {
    char line[24];
    OLED_Clear();
    OLED_DrawString(0,  0, "Dog Status");
    snprintf(line, sizeof(line), "P:%5.1f R:%5.1f", pitch, roll);
    OLED_DrawString(0, 12, line);
    snprintf(line, sizeof(line), "Action: %s", action);
    OLED_DrawString(0, 24, line);
    snprintf(line, sizeof(line), "Bat: %d%%", batt);
    OLED_DrawString(0, 36, line);
    OLED_Flush();
}
```

---

## 18. 视觉跟随（ESP32-CAM + OpenCV）

### 架构

```
ESP32-CAM → MJPEG 流 → PC(OpenCV) → HTTP POST → ESP32-Brain → UART → STM32
```

### Python 端（电脑侧）

```python
import cv2, requests

cap = cv2.VideoCapture("http://esp32-cam.local:81/stream")
while True:
    ret, frame = cap.read()
    if not ret: continue
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, (0, 120, 80), (10, 255, 255))
    M = cv2.moments(mask)
    if M["m00"] > 1000:
        cx = int(M["m10"] / M["m00"])
        w = frame.shape[1]
        dx = cx - w // 2
        if   dx < -30: cmd = "LEFT"
        elif dx >  30: cmd = "RIGHT"
        else:          cmd = "FORWARD"
        requests.post("http://esp32-brain.local/cmd", data={"cmd": cmd})
```

**简历话术**：基于 OpenCV 颜色空间识别实现目标跟随，支持机器狗自主追踪彩色目标。

---

## 19. 语音控制升级方案

| 方案 | 硬件 | 识别率 | 成本 | 说明 |
|------|------|--------|------|------|
| LD3320 | 额外模块 | 中 | 低 | 经典离线，50+ 命令词 |
| 幻尔 ASRPRO | 模块 | 高 | 中 | 图形化配置，效果好 |
| ESP32 + WakeNet | 无 | 中 | 免费 | 乐鑫官方，能离线唤醒 |
| 百度 / 讯飞 API | 需联网 | 高 | 云端 | 自然语言识别 |

**简历话术**：集成离线/云端双模语音识别，支持关键词唤醒与自然语言指令解析。

---

## 20. 完整项目实现路线（8 周）

| 周 | 目标 | 产出 |
|---|------|------|
| W1 | STM32 环境 + 点灯 + 串口 + PCA9685 | 单舵机转动 |
| W2 | 8 舵机联动 + 腿部机械标定 | 站立、蹲下 |
| W3 | 运动学逆解 + 平滑过渡 | 直线行走 |
| W4 | Trot 步态 + 前后左右 | 遥控走起来 |
| W5 | MPU6050 + 互补滤波 + PID | 抗推自稳 |
| W6 | FreeRTOS 重构 | 多任务并行 |
| W7 | ESP32 + MQTT + 手机网页 | 云端控制 |
| W8 | OLED + OTA + 视频录制 | 最终 demo |

---

## 21. 调试方法论

### 21.1 自下而上原则

1. **硬件层**：万用表量电压、示波器看 PWM、逻辑分析仪看 I2C。
2. **驱动层**：一次调一个驱动，串口打印确认读写寄存器正确。
3. **算法层**：先用硬编码坐标测试 IK 输出对不对。
4. **应用层**：拼装全链路，最后再联调。

### 21.2 打印大法

```c
printf("[%lu ms][IK] x=%.2f y=%.2f -> a1=%.2f a2=%.2f\r\n",
       HAL_GetTick(), x, y, *a1, *a2);
```

### 21.3 工具清单

- **示波器**：量 PWM 脉宽、WS2812 时序
- **逻辑分析仪**（Saleae 或廉价 USB 款）：看 I2C / UART 时序
- **串口助手**（XCOM / MobaXterm）：日志
- **STM32CubeMonitor**：实时变量监视
- **万用表**：基础电平检查

### 21.4 典型 Bug 速查表

| 现象 | 优先怀疑 |
|------|----------|
| I2C 卡死 | 上拉电阻、SDA/SCL 反接、地址错、电源未上 |
| 舵机乱转 | PWM 频率不是 50Hz、角度映射错、电源掉压 |
| 串口乱码 | 波特率、GND 未共地 |
| 蓝牙连不上 | 模块未上电、AT 波特率不一致（9600 vs 115200） |
| FreeRTOS HardFault | 栈深不够、任务中使用浮点未使能 FPU 保存 |
| WS2812 颜色错 | 数据顺序不是 GRB、时序误差大 |

---

## 22. 面试扩展 Q&A（衍生功能版）

### Q11：为什么用互补滤波而不用卡尔曼？

> 卡尔曼滤波效果更好但实现复杂、计算开销大；互补滤波只需一个系数 α，在 STM32F407 上运行成本低、调参简单、对姿态平衡已经够用。如果精度要求高可以升级到 MPU6050 内置 DMP 或卡尔曼滤波。

### Q12：PID 参数怎么调？

> 先只调 P：从小到大加，直到出现明显振荡但能跟踪目标；然后减半。再调 D 抑制振荡。最后加一点点 I 消除静差。工程上常用"试凑法" + "临界比例度法"。

### Q13：为什么用增量式 PID？

> 增量式输出的是"变化量"而不是绝对值，切换模式时不会突变，避免机械冲击；另外增量式不需要对误差做积分累加，抗积分饱和能力强。

### Q14：FreeRTOS 里 I2C 为什么要加互斥量？

> I2C 操作涉及 START → 地址 → 数据 → STOP 的原子序列，如果两个任务同时读 PCA9685 和 MPU6050，中断会交叉破坏时序，导致总线死锁。互斥量保证同时只有一个任务操作总线。

### Q15：MQTT 为什么比 HTTP 更适合 IoT？

> MQTT 是发布/订阅模型，长连接、消息小（最少 2 字节头）、支持 QoS 保证可达、遗嘱消息处理掉线。HTTP 每次请求都要握手、头开销大、不适合资源受限设备和高实时性场景。

---

**文档结束** · 祝面试顺利！
