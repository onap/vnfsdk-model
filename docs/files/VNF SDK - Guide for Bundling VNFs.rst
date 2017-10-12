VNF SDK - Guide for Bundling VNFs    
=================================


Step 1: Requirements
--------------------

1. Virtual machine images for each of your VDUs (likely in qcow2 format).
2. Scripts for installing and configuring each VDU per the standard TOSCA lifecycle workflows.
3. Optional: scripts for retrieving indicators from each VDU.
4. VNF SDK pkgtools

Step 2: Start with the provided VNF blueprint as a template
-----------------------------------------------------------

The VNF SDK comes with a "pingpong.yaml" template.

Copy the file and rename it as is appropriate for your VNF.

Step 3: Edit the VNF blueprint
------------------------------

You will need a node template for each VDU, which in turn has a "dependency" 
requirement on a host (a Compute node template).

The "dependency" requirement should have a relationship of type 
"vnfsdk.DependsOn".

There are two optional properties you can assign to this relationship:

1. "indicators" is a map of indicators. The keys are arbitrary, for you to name, 
  and depend on the specific capabilities of your VNF. These provide the link 
  between TOSCA and your indicator retrieval scripts.
2. "compute_dependencies" is a map of features you require the infrastructure to 
  provide for the compute node hosting your VDU. The keys are arbitrary, however 
  we will maintain a master list of names supported by various orchestrators 
  (TBD). Note that new names will likely be supported in the future as 
  technological innovation progresses, while deprecated ones will be ignored by 
  orchestrators. Likewise, names unsupported by a particular orchestrator will 
  be ignored.

Notes:

1. For "compute_dependencies", you have the option of specifying a single special key, "profile", that refers to a bundle of compute dependencies specified in 
  "compute-profiles.yaml". Optionally, you can override any of the dependencies 
  in the bundle by specifying them explicitly under "compute_dependencies".
  
2. By default, compute dependencies are considered to be absolute requirements 
  for your VNF to function. However, if your VNF can also function without them, 
  you can set "optional: true". This way the orchestrator will try to satisfy 
  the optional requirements, but will not report an error if it cannot.
3. You may optionally consider allowing operators to override your dependencies. 
  This is useful for allowing the VNF to run on various compute infrastructures 
  other than the defaults you specify, as long as they are indeed supported by 
  your VNF. In TOSCA, this is easily handled by declaring "inputs" for your 
  blueprint, with "default" values, and then using the "get_input" intrinsic 
  function to apply the provided value to any specific "compute_dependencies". 
  The "pingpong.yaml" has an example of such usage. The same technique can be 
  used to allow the operator to override your indicator scripts, and there is an 
  example of that as well.

Otherwise, this is a standard TOSCA blueprint using the Simple Profile for NFV, 
and you may extend the blueprint as is required.

Step 4: Bundle the VNF
----------------------

Arrange all your files (the TOSCA template, your scripts, and your virtual 
machine images) in the correct directory structure. File references provided in 
the blueprint must match the directory structure. If a script is referenced 
using a path with sub-directory "example" the initial directory must contain a 
directory called "example" with that script file inside. The directory structure 
will be kept inside the CSAR archive.

Run the bundling tool that is included in the VNF SDK.

The result will be a ".csar" file that can now be deployed by operators.

