# EC2 Blue/Green CloudFormation Deployments without DNS changes
This is a CloudFormation script to perform Blue/Green deployments using EC2 resources. It has been created to allow for fast switch over and switch back without having to wait for DNS propogation, or EC2 instance replacement. So instead of rotating instances in and out of a single target group, the target group that the live load balancer talks to is changed. This allows for a near instant switch between environments, and in the event of a problem, failback. In practice this seems to take around a second or so to actually switch. The non-live environment is created and deleted based on what phase of the deployment we are in.

There is also the staging load balancer that is connected to the non-live environment while it exists.

The phase is read by a variable in the SSM, that dicates which phase of the deployment you are in. The idea is that you can run a CloudFormation update without using multiple templates, you just change the SSM parameter each time. Alternatively you could pass it in on the command line. All resources are also managed by CloudFormation. 

There is an output defined with the DNS name of the staging load balancer, for those phases were it exists.

## Phases

The script is designed around 8 phases. 

1) Blue is live. No Green or staging resources created.
2) Blue is live. Green is spun up for staging, alongside a staging load balancer that points to the Green environment.
3) Blue is live. When testing is complete in Green, the staging load balancer is removed in preperation for pointing live to Green.
4) Green is live. The live load balancer listeners are repointed to the Green target groups. The Blue environment is still kept up in case of any issues following the switch over.
5) Green is live. the Blue environment is removed.
6) Green is live. Blue is spun up for staging, alongside a staging load balancer that points to the Blue environment.
7) Green is live. When testing is complete in Blue, the staging load balancer is removed in preperation for pointing live to Blue.
8) Blue is live. The live load balancer listeners are repointed to the Blue target groups. The Green environment is still kept up in case of any issues following the switch over.

The next step would be to go around to Phase 1.

If issues are found following the switch, you can move backwards to revert to the previous environment.

The idea is that something like Azure DevOps could also be used to update the SSM variable and run the CloudFormation script as part of the pipeline. This was created because CodeDeploy doesn't have enough options current around the time it takes for instances to be ready.
