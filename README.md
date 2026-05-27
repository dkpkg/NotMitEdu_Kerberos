# NotMitEdu_Kerberos

## Testing and updating imports

```sh
./dk0 update -f dk.u
```

## Testing and updating distribution scripts

```sh
./dk0 update -f dk.u

# CI tags use the package-native prerelease form `2.5.0-<timestamp>`.
# dk0 itself does not accept prerelease versions for `--library`, so local
# distribution commands use the dk-compatible form `2.5.<timestamp>` instead.

# Darwin
./dk0 -nosysinc --verbose distribute NotMitEdu_Kerberos-dist-Darwin_arm64 --library 'NotMitEdu_Kerberos@2.5.99991112223300' --actual-in-place dist-Darwin_arm64.u
./dk0 -nosysinc --verbose distribute NotMitEdu_Kerberos-dist-Darwin_x86_64 --library 'NotMitEdu_Kerberos@2.5.99991112223300' --actual-in-place dist-Darwin_x86_64.u

# Linux
./dk0 -nosysinc --verbose distribute NotMitEdu_Kerberos-dist-Linux_x86 --library 'NotMitEdu_Kerberos@2.5.99991112223300' --actual-in-place dist-Linux_x86.u
./dk0 -nosysinc --verbose distribute NotMitEdu_Kerberos-dist-Linux_x86_64 --library 'NotMitEdu_Kerberos@2.5.99991112223300' --actual-in-place dist-Linux_x86_64.u
./dk0 -nosysinc --verbose distribute NotMitEdu_Kerberos-dist-Linux_arm64 --library 'NotMitEdu_Kerberos@2.5.99991112223300' --actual-in-place dist-Linux_arm64.u
```

> [!IMPORTANT]
> Each target operating system updates its own object ids in the corresponding `dist-*.u/run.u` script.
> Run the matching operating system locally, or use the CI workflow in [`.github/workflows/distribute-2.5.yml`](.github/workflows/distribute-2.5.yml).

## Practical bundle checks

```sh
./dk0 --trust-local-package NotMitEdu_Kerberos -I etc/dk/v get-bundle NotMitEdu_Kerberos.Lookup@1.0.0 -d target/krblookup
./dk0 --trust-local-package NotMitEdu_Kerberos -I etc/dk/v get-bundle NotMitEdu_Kerberos.V5.Bundle@1.22.2 -d target/krb5src
```

## Using docker to update Linux distribution scripts

```sh
rm -rf t # or in PowerShell: Remove-Item -Force -Recurse .\t\

docker run -v .:/work --workdir /work -it quay.io/pypa/manylinux_2_28_x86_64:2026.04.08-5 sh ./dk0 -nosysinc -v -v distribute NotMitEdu_Kerberos-dist-Linux_x86_64 --library NotMitEdu_Kerberos@2.5.99991112223300 --actual-in-place dist-Linux_x86_64.u
docker run -v .:/work --workdir /work -it quay.io/pypa/manylinux_2_28_i686:2026.04.08-5 sh ./dk0 -nosysinc -v -v distribute NotMitEdu_Kerberos-dist-Linux_x86 --library NotMitEdu_Kerberos@2.5.99991112223300 --actual-in-place dist-Linux_x86.u
```

Linux arm64 distribution script updates require a native Linux arm64 host or CI runner.

## Updating dk0 and dk0.cmd scripts

On Windows PowerShell (from the root of this repository):

```powershell
$ErrorActionPreference = "Stop"

$scratch = ".\.local\dk0-updater"
Remove-Item $scratch -Recurse -Force -ErrorAction SilentlyContinue
git clone --depth 1 https://github.com/diskuv/dk.git $scratch

Copy-Item (Join-Path $scratch "dk0") -Destination ".\dk0" -Force
Copy-Item (Join-Path $scratch "dk0.cmd") -Destination ".\dk0.cmd" -Force

$dkVer = (Select-String -Path (Join-Path $scratch "dk0.cmd") -Pattern 'SET DK_VER=(.+)').Matches[0].Groups[1].Value.Trim()

Remove-Item $scratch -Recurse -Force

git commit -m "dk0 $dkVer" -- .\dk0 .\dk0.cmd
```
