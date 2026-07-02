## Description

`tnas` is powered by `TrueNas` (Community edition). It serves as a backup of the primary [NAS](NAS.md) instance.

## Hardware

- [ ] CPU: AMD Phenom(tm) II X4 965 Processor
- [ ] Memory: 12GB (the main board supports up to 16 GB (4 x 1.5V DDR3 DIMM sockets)
- [ ] Mainboard: GA-890XA-UD3 https://www.gigabyte.com/Motherboard/GA-890XA-UD3-rev-20#ov (2012)
- [ ] Graphics card:  Radeon HD 7950/8950 OEM / R9 280
- [ ] Price: €70
- [ ] PSU (brand new): MARSGAMING MPB650SI (€50)

## Upgrades

- [ ] June 2026: The PSU died in June 2026 and is replaced with a brand new one. At first, 
 a wrong version was bought (Mars Gaming MPB550SI, which only has 1 connector for PCIe 6+2 PIN).

 Fun fact: When I returned the wrong PSU to `Amazon`, I accidently packed up the only `IDE-to-SATA` power cable.
 Now I need to find a cheap one.!

## Dataflow

```
Primary Nas User ----> tNas  <---- control machine
```

Data is synchronized manually with `rsync` script.
