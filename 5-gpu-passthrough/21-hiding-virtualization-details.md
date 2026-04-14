# Hiding virtualization details

Some applications will refuse to open under a virtual machine when they can detect it. You can try to hide the fact that the system is running under a virtual machine by modifying your configuration.

Note that these steps are not guaranteed to work for all applications, but this is as far as you'll get without employing shady techniques. Still, this should be more or less good enough for most programs out there.

1.  Look for the `<os>` block, which will look something like:

    ```XML
    <os firmware="efi">
      ...
    </os>
    ```

    Append `<smbios mode="sysinfo"/>` to it:

    ```XML
    <os firmware="efi">
      ...
      <smbios mode="sysinfo"/>
    </os>
    ```

2.  After the `<os>` block, append the following:

    ```XML
    <sysinfo type="smbios">
      <bios>
        <entry name="vendor">...</entry>
        <entry name="version">...</entry>
        <entry name="date">...</entry>
        <entry name="release">...</entry>
      </bios>
      <system>
        <entry name="manufacturer">...</entry>
        <entry name="product">...</entry>
        <entry name="version">...</entry>
        <entry name="serial">...</entry>
        <entry name="uuid">...</entry>
        <entry name="sku">...</entry>
        <entry name="family">...</entry>
      </system>
      <baseBoard>
        <entry name="manufacturer">...</entry>
        <entry name="product">...</entry>
        <entry name="version">...</entry>
        <entry name="serial">...</entry>
        <entry name="asset">...</entry>
        <entry name="location">...</entry>
      </baseBoard>
      <chassis>
        <entry name="manufacturer">...</entry>
        <entry name="version">...</entry>
        <entry name="serial">...</entry>
        <entry name="asset">...</entry>
        <entry name="sku">...</entry>
      </chassis>
    </sysinfo>
    ```

    You will have to fill in all of the `...` parts with actual information. A couple notes:

    - You can copy the homework from your actual system by fetching SMBIOS information using `dmidecode` from the package of the same name, however...

    - You'll want to copy the UUID for `<system>` from your VM's UUID rather than your motherboard. Look for a `<uuid>` element at the top, near `<domain>`.

    - If you don't want to risk your hardware getting permabanned (in case you ever make a native installation of Windows again), it would be wise to fake the serial numbers as well.

3.  Look for a `<hyperv>` block (inside a `<features>` block), which will look something like:

    ```XML
    <hyperv mode="custom">
      ...
    </hyperv>
    ```

    Inside of it, append:

    ```XML
    <vendor_id state="on" value=""/>
    ```

4.  Below the `<hyperv>` block, add:

    ```XML
    <kvm>
      <hidden state="on"/>
    </kvm>
    ```

5.  Look for the `<cpu mode="host-passthrough">` block again. In it, append `<feature policy="disable" name="hypervisor"/>`, like:

    ```XML
    <cpu mode="host-passthrough" ...>
      ...
      <feature policy="disable" name="hypervisor"/>
    </cpu>
    ```

6.  Look for the `<clock offset="localtime">` block, which will look like:

    ```XML
    <clock offset="localtime">
      ...
    </clock>
    ```

    Replace it with;

    ```XML
    <clock offset="localtime">
      <timer name="rtc" present="no" tickpolicy="catchup"/>
      <timer name="pit" tickpolicy="discard"/>
      <timer name="hpet" present="no"/>
      <timer name="kvmclock" present="no"/>
      <timer name="hypervclock" present="yes"/>
      <timer name="tsc" present="yes" mode="native"/>
    </clock>
    ```

7.  Lastly, inside Windows, you will have to modify the `SystemBiosVersion` key in `HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System` in the registry and replace `BOCHS - 1` and `EDK II - 10000` with proper strings. You can take these strings from an existing baremetal Windows installation, if you have one; at worst, you can take them from the internet.

    Note that the `SystemBiosVersion` key resets everytime you boot Windows, so you will likely want to automate setting it on login. How to do this is left as an exercise to the reader.
