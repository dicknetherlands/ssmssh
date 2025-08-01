# ssmssh
1. Copy files to AWS EC2 using SCP over an AWS SSM connection using `ssmscp`
2. Open an SSH session over an AWS SSM connection using `ssmssh`

To run these commands you will need:

  1. Python3 installed on your local machine
  2. The AWS CLI (to use for `aws sso login`, etc.) installed and 
     AWS credentials with the correct permissions to 
     access the EC2 instance you want to copy files to/from
  3. An SSH public/private key pair. Your default pair at
     `~/.ssh/id_rsa` is fine. Use `ssh-keygen` to create one if  
     you don't have one

The `ssmscp` command is for copying data to/from an SSM connection. It
cannot be used to copy data between two EC2 instances using SSM.
(This could be a future improvement.)

The `ssmssh` command is used for opening an SSH session over SSM and doing
things like X forwarding and any other standard SSH operations.

## Step 1: Set up the EC2 instance to work with your local machine

You only need to set this up once per EC2 instance, per machine
that you want to copy data to/from, e.g. your laptop. It needs to
be done by each user intending to use this tool. Once set up
it remains so permanently and you can go straight to step 2 
next time.

`./ssmssh-setup --target <instance-id>`

It will use your default SSH public key, `~/.ssh/id_rsa.pub`. If you 
want to use a different key, you can specify it with `--publickey`
or `-k`.

The default user it will set up for is `ubuntu`. If you want a 
different username, e.g. `ec2-user`, specify it with `--username`
or `-u`.

## Step 2: Copy files to the EC2 instance using SCP!

`./ssmscp <source> <dest>`

**Either `source` or `dest` must be in the form of
`ubuntu@i-012345678:/path/to/file`, where `i-012345678` is your EC2
instance ID. The other path must be local.**

This runs a standard `scp` command. You can pass all the normal 
arguments to it as well as the source and dest, e.g. `-r` for a 
recursive copy.

Internally this will open and use temporary port 1234 on localhost. 
If you're using port 1234 for something else, specify a different 
port with `--port` or `-P`.

It'll use your default private SSH key, `~/.ssh/id_rsa`, unless told 
otherwise with the `--privatekey` or `-i` option. The key must match 
the public key you used with `ssmssh-setup` in Step 1 above.

# Step 3: SSH to the EC2 instance using SSM!

`./ssmssh ubuntu@<instanceid>`

where instanceid is e.g. `i-0123456578`.

All the same comments apply re: ports and keys as per `ssmscp` above.
It runs a standard `ssh` command and accepts all the same parameters.

**Note: If you want to forward X11 connections, all the standard
settings will work. Example: 
`DISPLAY=:0 ./ssmssh -X ubuntu@i-012345678`**

## Troubleshooting

 - The same chmod requirements apply to the SSH keys as for normal
ssh/scp. The private key must be `0600` and the public key must be
`0644`.

 - Make sure you specify the public key when running `ssmssh-setup`
and the private key when running `ssmscp`, if you're not using the
default key.

 - The instance must have the `sshd` daemon running and the `~/.ssh` 
folder must exist in the target user's home directory (e.g.
`/home/ubuntu/.ssh`), even if it is empty.

 - Instances do **NOT** need to have had an instance keypair assigned 
when first created, and even if they do you do **NOT** need to have
a copy of that keypair. 

 - Instances do **NOT** need to have port 22 open on their inbound 
security group, or any additional ports at all inbound or outbound. 

 - The instance requires the SSM agent installed and running and the 
role assigned to the instance must allow it to run SSM commands.

 - Your SSO profile must match a permissions set which allows you to 
access the instance and run SSM commands on it.

 - The instance will be searched for in the AWS account matching the 
SSO profile specified in the `AWS_PROFILE` environment variable.

