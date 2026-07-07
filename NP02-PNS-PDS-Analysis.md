## LArSoft analyzer PNS run at np02

```
# build on dunegpvm

/cvmfs/oasis.opensciencegrid.org/mis/apptainer/current/bin/apptainer shell --shell=/bin/bash \
-B /cvmfs,/exp,/nashome,/pnfs/dune,/opt,/run/user,/etc/hostname,/etc/hosts,/etc/krb5.conf --ipc --pid \
/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-dev-sl7:latest

export UPS_OVERRIDE="-H Linux64bit+3.10-2.17"
```

```
cd <your work directory in /exp/dune/app/users/your_username>
source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh

ups list -aK+ dunesw
# test release v10_20_09_01d00 featuring Geant4 11.4 (LArSoft v10_20_09_01)
# new G4 version with Nuclear De-excitation (NUDEX) package for neutron captures:
setup dunesw v10_20_09_01d00 -q e26:prof

mkdir dunesw_v10200901d00
cd dunesw_v10200901d00

mrb newDev
source <your work directory>/dunesw_v10200901d00/localProducts_larsoft_v10_20_09_01_e26_prof/setup

cd srcs
git clone https://github.com/weishi10141993/dunereco.git -b np02-vd-pns-pds-blipreco
# the custom branch was based on dunereco v10_20_09_01d00 following the test release version for G4 11.4
# ups list -aK+ dunereco
# mrb g -t v10_20_09_01d00 dunereco

mrb uc # if you have src code need to add to CMake

cd .. # top level, above srcs
mrbsetenv
mrbslp    # need this to proper config and fix wirecell error

setup ninja
mrb i --generator ninja

# this finishes set up the custom dunereco for np02 pds pns analysis
```

Copy all scripts (fcls and job scripts) and folders (mac files required for G4 neutron sim) from ```https://github.com/weishi10141993/VDPDSAna/tree/main/PNSCali/PDVD/fcl``` to ```/exp/dune/app/users/your_username>/dunesw_v10200901d00/``` directory (above srcs).


Produce sim samples
```
# event dump
lar -c eventdump.fcl <filename> -n 1
# fcl dump
fhicl-dump run_pdvd_blipana.fcl
```

Relogin:
```
/cvmfs/oasis.opensciencegrid.org/mis/apptainer/current/bin/apptainer shell --shell=/bin/bash \
-B /cvmfs,/exp,/nashome,/pnfs/dune,/opt,/run/user,/etc/hostname,/etc/hosts,/etc/krb5.conf --ipc --pid \
/cvmfs/singularity.opensciencegrid.org/fermilab/fnal-dev-sl7:latest

export UPS_OVERRIDE="-H Linux64bit+3.10-2.17"

source /cvmfs/dune.opensciencegrid.org/products/dune/setup_dune.sh
setup dunesw v10_20_09_01d00 -q e26:prof
source <your work directory in /exp/dune/app/users/your_username>/dunesw_v10200901d00/localProducts_larsoft_v10_20_09_01_e26_prof/setup

mrbsetenv
mrbslp
```

Recompile:
```
cd ${MRB_BUILDDIR}  
mrbsetenv
mrbslp     # required to configure wirecell properly
ninja install
```

Jobs:
```
tar -cvf np02pnsAna.tar --exclude='./build_slf7.x86_64' --exclude='./srcs' .

# in the same directory there is justin_profile.sh
source justin_profile.sh

sh launchscript_localdirectory.sh
```
