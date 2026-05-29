# baremetal-sw
Miscellaneous C tests for Chipyard

## Example command to program chip id:
``make run-binary CONFIG=DoubleRoutingChipletConfig BINARY=../../../baremetal-sw/build/router.riscv EXTRA_SIM_FLAGS="+chip_id0=0x00004080:0x00000001 +chip_id1=0x00004080:0x00000002"``

## Configs

### router.c
```
class ComputeChipletRouterConfig extends Config(
  new chipyard.harness.WithD2DLoopback ++
  new chipyard.iobinders.WithD2DPunchthrough ++
  new testchipip.soc.WithChipletRouting(testchipip.soc.ChipletRoutingParams(
    routerParams = testchipip.soc.OffchipRouterParams(tableEntries = 4),
    ports = Seq(new testchipip.ctc.CTCParams(phyParams = None))
  )) ++
  new chipyard.RocketConfig
)

class SertlComputeChipletConfig extends Config(
  new testchipip.soc.WithOffchipAddressRange(AddressSet.misaligned(0x200000000L, 0x400000000L)) ++
  new testchipip.soc.WithD2DPorts(Seq(testchipip.serdes.SerialTLParams(
      client = Some(testchipip.serdes.SerialTLClientParams(masterWhere = SBUS)),
      manager = Some(testchipip.serdes.SerialTLManagerParams()),
      phyParams = testchipip.serdes.CreditedSourceSyncSerialPhyParams()
      //bundleParams = testchipip.serdes.TLSerdesser.STANDARD_TLBUNDLE_PARAMS.copy(sourceBits = 9) // Temp hack
    ))) ++
  new ComputeChipletRouterConfig
)

class DoubleRoutingChipletConfig extends Config(
  new chipyard.harness.WithAbsoluteFreqHarnessClockInstantiator ++
  new chipyard.harness.WithMultiChipD2D(chip0=1, chip1=0, chip0portId=0, chip1portId=0) ++ 
  new chipyard.harness.WithMultiChip(0, new SertlComputeChipletConfig) ++
  new chipyard.harness.WithMultiChip(1, new SertlComputeChipletConfig) 
)
```

### router-translate.c
```
class Router2xConfig extends Config(
  new chipyard.iobinders.WithD2DPunchthrough ++
  new testchipip.soc.WithChipletRouting(testchipip.soc.ChipletRoutingParams(
    routerParams = testchipip.soc.OffchipRouterParams(tableEntries = 4),
    ports = Seq(
      testchipip.serdes.SerialTLParams(
        client = Some(testchipip.serdes.SerialTLClientParams(masterWhere = SBUS)),
        manager = Some(testchipip.serdes.SerialTLManagerParams()),
        phyParams = testchipip.serdes.CreditedSourceSyncSerialPhyParams()
      ),
      testchipip.serdes.SerialTLParams(
        client = Some(testchipip.serdes.SerialTLClientParams(masterWhere = SBUS)),
        manager = Some(testchipip.serdes.SerialTLManagerParams()),
        phyParams = testchipip.serdes.CreditedSourceSyncSerialPhyParams()
      )
    )
  )) ++
  new chipyard.RocketConfig
)

class RouterTranslationConfig extends Config(
  new chipyard.harness.WithAbsoluteFreqHarnessClockInstantiator ++
  new chipyard.harness.WithMultiChipD2D(chip0=1, chip1=0, chip0portId=0, chip1portId=0) ++ 
  new chipyard.harness.WithMultiChipD2D(chip0=1, chip1=0, chip0portId=1, chip1portId=1) ++ 
  new chipyard.harness.WithMultiChip(0, new Router2xConfig) ++
  new chipyard.harness.WithMultiChip(1, new Router2xConfig) 
)
```

### router-bypass.c
```
class RouterBypassConfig extends Config(
  new chipyard.harness.WithD2DTestRAM ++
  new chipyard.iobinders.WithD2DPunchthrough ++
  new testchipip.soc.WithChipletRouting(testchipip.soc.ChipletRoutingParams(
    routerParams = testchipip.soc.OffchipRouterParams(tableEntries = 4),
    ports = Seq(
      new testchipip.ctc.CTCParams(offchip = Seq(AddressSet(0x100000000L, 0xFFFFF)), phyParams = Some(testchipip.ctc.CTCMemSerialPhyParams(offchip = Seq(AddressSet(0x100000000L, 0xFFFFF))))),
      new testchipip.ctc.CTCParams(offchip = Seq(AddressSet(0x200000000L, 0xFFFFF)), phyParams = Some(testchipip.ctc.CTCMemSerialPhyParams(offchip = Seq(AddressSet(0x200000000L, 0xFFFFF)))))
    )
  )) ++
  new chipyard.RocketConfig
)
```
