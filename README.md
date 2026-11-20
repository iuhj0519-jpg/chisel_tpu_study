# LZero RTL Project

고려대학교 컴퓨터 아키텍처 동아리 **LZero** 활동에서 작성한 Chisel 기반 NPU RTL 저장소이다.
NPU custom instruction의 `rs1` decoding, global stall 생성·분배, TPU-style systolic array MatMul 연산기를 다룬다.

## 디렉터리

| 경로 | 역할 |
|---|---|
| `spec/isa/` | NPU custom instruction operand(`rs1`, `rs2`) 정의 |
| `src/main/scala/pipeline/` | `rs1` decoder, stall generator, stall distributor |
| `src/main/scala/tpu/` | MAC PE, MXU, Accumulator, Data Orchestrator 기반 MatMul 연산기 |
| `src/test/scala/tpu/` | chiseltest 기반 MatMul testbench |

## Spec

- [NPU Custom Instruction Operand Spec](spec/isa/ISA_spec.md): `rs1` 64-bit bit allocation map, `hw_enables` 세부 구성, `rs2` `npu_ctrl` struct

## Module

### `pipeline/`

| 파일 | Module | 역할 |
|---|---|---|
| `Rs1_Decoder.scala` | `NpuRs1Decoder` | `rs1`에 담긴 이번 작업의 설정값을 field별로 분리한다. 분리된 값의 저장과 TPU·VPU·DMA 전달은 상위 Control Unit이 담당한다. |
| `Stall_Generator.scala` | `StallGenerator` | OCM Controller와 DMA Write Controller의 impending 신호를 모아 Compute Unit 전체에 배포할 global stall을 생성한다. |
| `Stall_Distributor.scala` | `StallDistributor` | impending 입력의 OR 결과를 `rootStallReg`(1 cycle)와 upper/lower branch register(2 cycle)를 거쳐 tree 구조로 분배한다. |

### `tpu/`

| 파일 | 주요 Module | 역할 |
|---|---|---|
| `MatMul_Operator.scala` | `MyMacUnit`, `Accumulator16`, `MyMXU_with_Accumulator` | 단순 MatMul 연산만 수행하는 연산기 |
| `MatMul_DataOrchestrator.scala` | `MyMacUnit`, `DataOrchestrator`, `MyMXU`, `Accumulator16`, `MyTPU_Complete_Top` | MatMul 연산과 input matrix skewing을 함께 수행하는 16×16 구조 |

- Activation 8-bit, partial sum 16-bit의 `SystolicIO` interface를 사용한다.
- 두 파일이 `SystolicIO`, `MyMacUnit` 등 같은 이름의 class를 각각 정의하므로 한 compile 단위에 함께 넣으면 이름이 충돌한다.

## Test

`src/test/scala/tpu/MatMul_DataOrchestrator_Test.scala`는 chiseltest와 ScalaTest(`AnyFlatSpec`)로 작성되었으며 `WriteVcdAnnotation`으로 VCD 파형을 남긴다.
이 저장소에는 `build.sbt` 등 build 설정 파일이 포함되어 있지 않다.

## 관련 저장소

- [lzero_01_rtl](https://github.com/iuhj0519-jpg/lzero_01_rtl): LZero 동아리 RTL 트랙 원본 저장소(`l-zero-society/2026-1-edge-llm-npu-rtl`)의 fork
