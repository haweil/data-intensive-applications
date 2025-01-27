
## Chapter 1: Reliable, Scalable, and Maintainable Applications

## Pages 7 to 13

## 1. Hardware Faults
### Summary
Hardware faults are a common cause of system failure, including issues like hard disk crashes, faulty RAM, power outages, or disconnected network cables. In large datacenters, such failures occur frequently due to the sheer number of machines. Traditional solutions focus on adding redundancy to individual hardware components, such as RAID configurations, dual power supplies, and backup power systems. While this approach can keep machines running for years, it is insufficient for modern applications that use many machines, as hardware failure rates increase proportionally with scale. To address this, there is a shift toward software fault-tolerance techniques, allowing systems to withstand entire machine failures. Such systems also enable operational advantages like rolling upgrades without planned downtime.

### Key Points
1. **Hardware Failures**:
   - Common issues: disk crashes, faulty RAM, power outages, and network errors.
   - Frequent in large datacenters due to scale.

2. **Redundancy Techniques**:
   - Use of RAID, dual power supplies, and backup power systems.
   - Redundant components minimize failure impact.

3. **Scalability Challenges**:
   - Increased machine use raises the frequency of hardware faults.
   - Cloud platforms like AWS prioritize flexibility over single-machine reliability.

4. **Software Fault-Tolerance**:
   - Enables systems to tolerate machine failures.
   - Supports operational improvements like rolling upgrades.

