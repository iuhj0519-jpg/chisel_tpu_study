# NPU Custom Instruction Operand Spec

기존 `ISA_spec_1.png`, `ISA_spec_2.png`의 표를 Markdown으로 옮긴 문서이다.
손글씨 정리본은 [`ISA_visualization.jpg`](ISA_visualization.jpg)에 있다.

## `rs1` Bit Allocation Map

| Bit Range | Field Name | Size | Description |
|---|---|---|---|
| 63 | `fusion_en` | 1b | 1: TPU → VPU1 → VPU2 End-to-End 라우팅 및 8-tile 출력 제어 |
| 62:58 | `lut_write` | 5b | [4: Act, 3: Exp, 2: Scale, 1: Sin, 0: Cos] LUT/Param 기록 펄스 |
| 57:51 | `hw_enables` | 7b | 연산 유닛 동작 모드 및 Enable 플래그 묶음 |
| 50:49 | `transpose_en_rd` | 2b | DMA Read 시 입력 Transposer 통과 여부 [1: W_trans, 0: X_trans] |
| 48 | `transpose_en_wr` | 1b | 1: DMA Write 시 종단 Transposer 통과 (0: Bypass) |
| 47 | `tile_strided_rd` | 1b | 1: Read 시 Struct에 정의된 `input_stride` 적용 |
| 46 | `tile_strided_wr` | 1b | 1: Write 시 Struct에 정의된 `output_stride` 적용 |
| 45:44 | `input_point` | 2b | 입력 스트림 시작점 (`00`: MXU, `01`: VPU1, `10`: VPU2) |
| 43:42 | `out_point` | 2b | 출력 스트림 종착점 (`00`: VPU2, `01`: VPU1) |
| 41:38 | `valid_row_count` | 4b | Edge Tile의 유효 Row 개수 (0x0: 16개 전체 유효, 0x1~0xF: 해당 개수) |
| 37:34 | `valid_col_count` | 4b | Edge Tile의 유효 Col 개수 (0x0: 16개 전체 유효, 0x1~0xF: 해당 개수) |
| 33 | `size_embed` | 1b | 1: 아래 32비트 차원 정보 사용, 0: `rs2` Struct의 차원 정보 사용 |
| 32:22 | `out_rowNum` | 11b | Output M 차원 크기 (1/16 Scaled, 최대 32,768) |
| 21:11 | `out_intermNum` | 11b | 연산 K 차원 크기 (1/16 Scaled, 최대 32,768) |
| 10:0 | `out_colNum` | 11b | Output N 차원 크기 (1/16 Scaled, 최대 32,768) |

### `hw_enables` (57:51) 세부 구성

[`Rs1_Decoder.scala`](../../src/main/scala/pipeline/Rs1_Decoder.scala) 기준이다.

| Bit | Signal | Size | Description |
|---|---|---|---|
| 57 | `mxuEn` | 1b | MXU Enable |
| 56:55 | `aluMode` | 2b | `00`: bypass, `01`: add, `10`: mul, `11`: reserved |
| 54 | `actEn` | 1b | Activation Enable |
| 53:52 | `normMode` | 2b | `00`: bypass, `01`: RMSNorm, `10`: LayerNorm, `11`: reserved |
| 51 | `ropeEn` | 1b | RoPE Enable |

## `rs2`: `npu_ctrl_addr`

`ISA_visualization.jpg`에 정리된 `rs2` Struct이다.

```c
typedef struct {
    void* input_addr, weight1_addr, weight2_addr;
    void* act_lut_addr, exp_lut_addr, scale_lut_addr, rope_sin_addr, rope_cos_addr;
    void* output_addr, norm_buff_addr;
    unsigned out_rowNum, out_intermNum, out_colNum;  // 1/16 scale
} npu_ctrl;

// size: 8 x 13 byte = 104 bytes
```
