# Virtualization-Assignment-3
To add support for 0x4ffffffe and 0x4fffffff I have modified the cpuid.c and vmx.c. The SDM defines Exit reasons on Volume 3 Appendix C VMX Basic Exit Reasons section. Here we find all the exit reasons defined by Intel SDM. The Exit reasons supported by VMX are mentioned in file vmx.h at linux/arch/kvm.vmx/vmx.h
First we list the exit reasons not supported by SDM and create a separate list for exit reasons not supported by KVM. The functions to validate the exit reason in modified cpuid code is shown below,

![01_exit_reason_check_function](https://user-images.githubusercontent.com/85700971/206966923-a21961a8-eeba-4fd2-81c1-bfabd5a1fca8.png)

Once we verify that the subleaf provided to cpuid program is a valid exit reason, we proceed to increase the count and the timer for the particular exit reason in the defined arrays. 

To add support for CPUID  leaf nodes  0x4ffffffe and 0x4fffffff please checkout the repo 

 git clone https://github.com/hegdebhavya/linux.git 

Changes to add support for the new leafnodes can be seen here <<commit link>>
Next we build the modules by running the commands

```
sudo make modules
```
We then install the modules

```
sudo make INSTALL_MOD_STRIP=1 modules_install

```

Please reload the modules by following the below commands

```
rmmod kvm_intel
rmmod kvm
modprobe kvm_intel
modprobe kvm

```

Next, we launch the 32-bit Ubuntu VM using virt-manager
```
virt-manager
```

The output for testing the number of exits for exit reason 10 (0x0a) which is EXIT_REASON_CPUID can be checked by running command below,
```
./cpuid -s 0x4ffffffe -s 10 
```
The output can be seen in the eax register in screenshot below,
![02_output_0x4ffffffe](https://user-images.githubusercontent.com/85700971/206967270-dabe81bd-ff89-4003-a599-8f8b8cb82236.png)

We observe that the total count for exit 10 is increasing everytime we run the cpuid command.
The dmesg for this exit can be seen in screenshot below

![03_dmesg_0x4ffffffe](https://user-images.githubusercontent.com/85700971/206967332-9da28612-0635-41f8-a7f8-606d588f8951.png)

The output for testing the total time needed for processing exit reason 10 (0x0a) which is EXIT_REASON_CPUID can be checked by running command below,

```
./cpuid -s 0x4fffffff -s 10
```

For output, the high 32-bits of total time spent are returned in register ebx and the low 32-bit of the total time spent are returned in the register ecx.

The output in the ebx, ecx registers can be seen in the screenshot below,

![04_output_0x4fffffff](https://user-images.githubusercontent.com/85700971/206967438-c4d847e4-6d18-4238-884f-7b15cca14dd6.png)

We observe that the total time for exit 10 is increasing everytime we run the cpuid command; this is seen in registers ebx and ecx.

Next, we test the output of cpuid when we give exit reason which is not supported by SDM, we can test this by using Exit reason 77

![05_unsupported_SDM](https://user-images.githubusercontent.com/85700971/206967476-632dba39-f14c-448f-bd52-9cfbde198e1c.png)

Next, we test the output of cpuid when we give exit reason which is not supported by KVM but is supported by SDM, we can test this by using Exit reason 5

![06_unsupported_KVM](https://user-images.githubusercontent.com/85700971/206967530-32960422-207d-489b-937f-7f1dfb3d2f48.png)

The output contains eax, ebx, ecx and edx as zero.

### Questions


3. Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there more exits performed during certain VM operations? Approximately how many exits does a full VM boot entail?

Ans.
Yes I see IO_INSTRUCTION (exit reason 30) exit increase at a stable rate. I also observed CPUID (exit reason 10) exit increasing at stable rate while running the cpuid instructions. 

To calculate the number of exits for full VM boot we can check this by running reboot command and then checking cpuid for leafnode 0x4ffffffc, this can be seen below,

![08_boot_exits](https://user-images.githubusercontent.com/85700971/206967645-829abae7-a4ec-4dbe-b87e-e0ead7256a8e.png)

We observe that the reboot  caused 3,334,075 (0x0032dfbb) total exits.


4. Of the exit types defined in the SDM, which are the most frequent? Least?
Ans.
I collected the output of cpuid command for exit reasons from 0 - 75 and found the following values in the counter


I found Exit reason 48 (EPT_VIOLATION) and Exit reason 30 (IO_INSTRUCTION) to be most frequent. 
The Exit reason 29 (DR_ACCESS) is the least frequent.

![07_checking_frequency](https://user-images.githubusercontent.com/85700971/206967722-0f378bc5-039a-4a2e-ba4b-0c292c3709ee.png)







