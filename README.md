


Single VM test
===============

1. Install fio:

       # apt install fio


2. Attach a test disk.

   Create a test disk and attach it to the test VM. Recommended disk size 100 GB or larger. 

   List the available block devices with:

        # lsblk

3. Update the fio job description files with the test disk path. For example, if the test disk is /dev/vdb run:

        # sed -i '/^filename=/ c filename=/dev/vdb' [0-9][0-9]_*

  Verify the result:

        # egrep ^filename= *

4. Fill the test drive

  Before running fio tests, fill the test drive with random data. Running fio tests on non-populated devices can provide false results.

        # fio 00_fill

4. Run fio test jobs. 

  Run a selected test job. For example:

        # fio 01_lat-r-4k-1



Run parallel tests from multiple VMs
=====================================

One VM may not generate sufficient test traffic to measure the storage system's performance. Some tests may require multiple parallel test jobs running from multiple VMs. These are typically IOPS and bandwidth tests.

1. Create test VMs. 

  Depending on the storage system, you may need to run between 3 and 20 or more VMs. Each test VM shall have one test drive attached. The test drives shall be with the same size and attached as the same device name - e.g., /dev/vdb.

2. Install and start the fio service.

  On each test VM, install fio.service

        # apt install fio
        # cp fio.service /etc/systemd/system && systemctl enable --now fio.service

  Make sure the selected fio service port (tcp 8765) is open in the firewall.

3. Prepare a control VM.

  Use one of the test VMs as the control VM. Copy this directory to the control VM.

  On the control VM, create a list of the test VM IP addresses:

        # cat vm_list
        10.0.0.3
        10.0.0.4
        10.0.0.5
        10.0.0.6
        10.0.0.7

  The control VM can also be a test VM, so include its IP address in the list too.

  On the control VM, update the fio job description files with the test disk path. For example, if the test disk is /dev/vdb run:

        # sed -i '/^filename=/ c filename=/dev/vdb' [0-9][0-9]_*

  Note: This device name will be used on all test VMs. Make sure the test disk has the same name on all test VMs.

4. Fill the test drives.

   On the control VM run:

        # fio --client vm_list 00_fill

5. Run the parallel test.

  On the control VM, run the selected fio test:

        # fio --client vm_list 03_rand-r-4k-64

  The report will include the aggregated result from all test VMs.

