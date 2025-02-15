---
title: Loading and Unloading
description: Loading and Unloading
keywords:
- filter manager WDK file system minifilter , loading/unloading drivers
- minifilter drivers WDK , driver loading
- file system minifilter drivers WDK , driver loading
- driver loading WDK file system
- loading drivers WDK file system
- unloading drivers
ms.date: 10/16/2019
---

# Loading and Unloading

## About Loading and Unloading

A minifilter driver can be loaded at any time while the system is running. If a minifilter driver's [INF file](creating-an-inf-file-for-a-minifilter-driver.md) specifies a driver start type of SERVICE_BOOT_START, SERVICE_SYSTEM_START, or SERVICE_AUTO_START, the minifilter driver is loaded according to existing load order group definitions for file system filter drivers, to support interoperability with legacy filter drivers. While the system is running, a minifilter driver can be loaded through a service start request (sc start, net start, or the service APIs), or through an explicit load request (fltmc load, [**FltLoadFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltloadfilter), or [**FilterLoad**](/windows/win32/api/fltuser/nf-fltuser-filterload)).

A minifilter driver's [**DriverEntry**](writing-a-driverentry-routine-for-a-minifilter-driver.md) routine is called when the minifilter driver is loaded, so the minifilter driver can perform initialization that will apply to all instances of the minifilter driver. Within its **DriverEntry** routine, the minifilter driver calls [**FltRegisterFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltregisterfilter) to register callback routines with the filter manager and [**FltStartFiltering**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltstartfiltering) to notify the filter manager that the minifilter driver is ready to start attaching to volumes and filtering I/O requests.

Minifilter driver instances are defined in the [INF file](creating-an-inf-file-for-a-minifilter-driver.md) used to install the minifilter driver. A minifilter driver's INF file must define a default instance, and it can define additional instances. These definitions apply across all volumes. Each instance definition includes the instance name, its altitude, and flags that indicate whether the instance can be attached automatically, manually, or both. The default instance is used to order minifilter drivers so that the filter manager calls the minifilter driver's mount and instance setup callback routines in the correct order. The default instance is also used with explicit attachment requests when the caller doesn't specify an instance name.

The filter manager automatically notifies a minifilter driver about an available volume by calling its [*InstanceSetupCallback*](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_setup_callback) routine on the first create operation after the volume is mounted. This can occur before [**FltStartFiltering**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltstartfiltering) returns, when the filter manager enumerates existing volumes at system startup. It can also occur during runtime, when a volume is mounted or as a result of an explicit attachment request (fltmc attach, [**FltAttachVolume**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltattachvolume), or [**FilterAttach**](/windows/win32/api/fltuser/nf-fltuser-filterattach)).

A minifilter driver instance is torn down when the minifilter driver is unloaded, the volume the instance is attached to is being dismounted, or as a result of an explicit detach request (fltmc detach, [**FltDetachVolume**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltdetachvolume), or [**FilterDetach**](/windows/win32/api/fltuser/nf-fltuser-filterdetach)). If the minifilter driver registers an [*InstanceQueryTeardownCallback*](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_query_teardown_callback) routine, it can fail an explicit detach request by calling **FilterDetach** or **FltDetachVolume**. The teardown proceeds as follows:

- If the minifilter driver registered an [*InstanceTeardownStartCallback*](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_teardown_callback) callback routine, the filter manager calls it at the beginning of the teardown process. In this routine, the minifilter driver should complete all pending operations, cancel or complete other work such as I/O requests generated by the minifilter driver, and stop queuing new work items.

- During instance teardown, any currently executing preoperation or postoperation callback routines continue normal processing, any I/O request that is waiting for a postoperation callback can be "drained" or canceled, and any I/O requests generated by the minifilter driver continue normal processing until they are complete.

- If the minifilter driver registered an [*InstanceTeardownCompleteCallback*](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_teardown_callback) routine, the filter manager calls this routine after all outstanding I/O operations have been completed. In this routine, the minifilter driver closes any files that are still open.

- After all outstanding references to the instance are released, the filter manager deletes remaining contexts and the instance is completely torn down.

While the system is running, a minifilter driver can be unloaded through a service stop request (sc stop, net stop, or the service APIs), or through an explicit unload request (fltmc unload, [**FltUnloadFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltunloadfilter), or [**FilterUnload**](/windows/win32/api/fltuser/nf-fltuser-filterunload)). It should be noted that the filter will be unloaded regardless of any dependencies registered to the SCM.

A minifilter driver's [*FilterUnloadCallback*](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_filter_unload_callback) routine is called when the minifilter driver is unloaded. This routine closes any open communication server ports, calls [**FltUnregisterFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltunregisterfilter), and performs any needed cleanup. Registering this routine is optional. However, if the minifilter driver does not register a *FilterUnloadCallback* routine, the minifilter driver cannot be unloaded. For more information about this routine, see [Writing a FilterUnloadCallback Routine](writing-a-filterunloadcallback-routine.md).

## Filter Manager Routines for Loading and Unloading Minifilter Drivers

The filter manager provides the following support routines for explicit load and unload requests, which can be issued from user mode or kernel mode:

- [**FilterLoad**](/windows/win32/api/fltuser/nf-fltuser-filterload)

- [**FilterUnload**](/windows/win32/api/fltuser/nf-fltuser-filterunload)

- [**FltLoadFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltloadfilter)

- [**FltUnloadFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltunloadfilter)

The following routines are used to register and unregister callback routines for instance setup and teardown:

- [**FltRegisterFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltregisterfilter)

- [**FltStartFiltering**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltstartfiltering)

- [**FltUnregisterFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltunregisterfilter)

## Minifilter Driver Callback Routines for Instance Setup, Teardown, and Unload

The following minifilter driver callback routines are stored as members of the [**FLT_REGISTRATION**](/windows-hardware/drivers/ddi/fltkernel/ns-fltkernel-_flt_registration) structure that is passed as a parameter to [**FltRegisterFilter**](/windows-hardware/drivers/ddi/fltkernel/nf-fltkernel-fltregisterfilter):

| Callback Routine Member Name | Callback Routine Type |
| --------------------- | --------------------- |
| **InstanceSetupCallback** | [PFLT_INSTANCE_SETUP_CALLBACK](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_setup_callback) |
| **InstanceQueryTeardownCallback** | [PFLT_INSTANCE_QUERY_TEARDOWN_CALLBACK](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_query_teardown_callback) |
| **InstanceTeardownStartCallback** | [PFLT_INSTANCE_TEARDOWN_CALLBACK](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_teardown_callback) |
| **InstanceTeardownCompleteCallback** | [PFLT_INSTANCE_TEARDOWN_CALLBACK](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_instance_teardown_callback) |
| **FilterUnloadCallback** | [PFLT_FILTER_UNLOAD_CALLBACK](/windows-hardware/drivers/ddi/fltkernel/nc-fltkernel-pflt_filter_unload_callback)
