> [← Documentation home](../README.md) · [Simple API](simple-auth.md) · [Bedrock](bedrock.md) · [Nightflyer](nightflyer.md) · [Management API](management-api.md) · [Server-side Variables](variables.md) · **SL-HWID** · [Quicksilver](quicksilver.md)

## SL-HWID

SL-HWID is a fault-tolerant, cross-platform hardware identifier for C++20 and .NET 8 applications. Instead of treating one serial number as the whole machine, it derives a random device identifier from a set of hardware and software signals. That lets ordinary changes such as replacing a monitor leave the identifier intact, while making a single-value spoof less useful.

It is useful with System Locker and with other licensing systems. In either case, send the generated value to your authorization service and call `commit` only after that service accepts the machine. The commit re-centers the local recovery data around any ordinary hardware drift.

The source, license, and full cross-platform notes are available in the [SL-HWID repository](https://github.com/systemlocker/SL-HWID).

### Before you start

SL-HWID stores local recovery data. On Windows, it uses `HKLM\SOFTWARE\SystemLocker` when it can write there, and otherwise uses the current user's registry hive. Do not delete that state as part of normal cleanup. Deleting it, or deliberately using `ForceReenroll` / `forceReenroll`, creates a new HWID that needs a server-side device reset or reactivation.

### Visual Studio: C++20

The C++ library is a CMake project with no third-party dependencies. Use Visual Studio 2022 with the **Desktop development with C++** workload and CMake 3.20 or later.

1. Clone or download [SL-HWID](https://github.com/systemlocker/SL-HWID) into your project, for example at `external/SL-HWID`.
2. Open your CMake project folder in Visual Studio.
3. Add the library to your top-level `CMakeLists.txt`, then let Visual Studio reload CMake.
4. Build your application from **Build &gt; Build All**.

```cmake
cmake_minimum_required(VERSION 3.20)
project(LicenseApp LANGUAGES CXX)

add_subdirectory(external/SL-HWID/cpp)

add_executable(LicenseApp main.cpp)
target_link_libraries(LicenseApp PRIVATE SLHwid::slhwid)
target_compile_features(LicenseApp PRIVATE cxx_std_20)
```

Prepare an HWID before authorization. Commit it only after the response authorizes this device.

```cpp
#include <slhwid/slhwid.hpp>

void authorizeCurrentDevice()
{
    auto session = slhwid::prepare({});
    if (!session) {
        showAccessError(session.error().message);
        return;
    }

    const auto response = authorizeWithYourService(session->hwid());
    if (!response.authorized) {
        showAccessError(response.message);
        return;
    }

    session->commit();
    grantAccess();
}
```

### Visual Studio: C# / .NET

The .NET library targets .NET 8 and has no NuGet dependencies. Install the **.NET desktop development** workload in Visual Studio 2022.

1. Clone or download [SL-HWID](https://github.com/systemlocker/SL-HWID) into your solution, for example at `external/SL-HWID`.
2. Open your solution in Visual Studio.
3. Right-click your application project, choose **Add &gt; Project Reference**, choose **Browse**, and select `external/SL-HWID/csharp/SLHwid/SLHwid.csproj`.
4. Confirm that your application targets `net8.0` or a compatible framework, then build the solution.

You can also add the reference directly to your project file:

```xml
<ItemGroup>
  <ProjectReference Include="..\external\SL-HWID\csharp\SLHwid\SLHwid.csproj" />
</ItemGroup>
```

```csharp
using SLHwid;

async Task AuthorizeCurrentDeviceAsync()
{
    SLHwidSession session;
    try
    {
        session = SLHwid.Prepare(new SLHwidOptions());
    }
    catch (SLHwidDriftException)
    {
        ShowAccessError("This device needs to be reactivated.");
        return;
    }
    catch (SLHwidCorruptDataException)
    {
        ShowAccessError("This device needs to be reactivated.");
        return;
    }

    var response = await AuthorizeWithYourServiceAsync(session.Hwid);
    if (!response.Authorized)
    {
        ShowAccessError(response.Message);
        return;
    }

    session.Commit();
    GrantAccess();
}
```

### Using SL-HWID with Bedrock

Bedrock clients for .NET, Go, Node.js, and Python use SL-HWID by default from version 1.0.0. In the Bedrock C++ client, set the `hwid` configuration value to an empty string to opt in. See the [Bedrock reference](bedrock.md) for configuration and migration guidance. If you are moving an existing System Locker system to SL-HWID, reset existing HWID claims before customers update so the new identifier can be accepted.
