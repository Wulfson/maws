## MAWS

This project will be my AWS tools for creating a Minecraft server (thus, MAWS).

The initial form is just an assortment of scripts that I've been using to create my instance for a while, and finally got unnerved enough about them being unrevisioned that I'm checking them into a repository. As such, I expect they won't be terribly useful to anyone else yet...but I'll document how to make them work, regardless. Note that I'll be working entirely with the Linux CLI; if you want to use a GUI for any of this, you'll need to figure it out on your own. Also note that the current documentation assumes that you have good working knowledge of how to use AWS. After I get the tools into a more user-friendly state, I'll create a guide to explain the whole process.

### WARNING

This whole process makes assumptions that certain things exist in your AWS account. Since I haven't supplied those things, it is highly likely that following these instructions will result in an instance that does not work. I am providing these instructions strictly to document the process I currently use. I believe it should be possible to figure out what needs done by studying the instructions and source provided, but I do not believe it will be especially obvious, and those capable of doing so probably didn't actually need the scripts in the first place.

Additionally, AWS has no problem charging you while you play around with things. If you spin up a number of broken instances and forget to turn them off, you could be facing a steep bill.

#### Setup

You'll have to start out by installing the AWS CLI Tools, like so:

```shell
apt-get update && apt-get upgrade -y
apt-get -q -y install python-pip python-dev build-essential
pip install awscli
```

After that, you'll need a file with your AWS credentials available for the AWS tools to use somewhere, which will look something like:

```ini
[default]
aws_access_key_id = ASTRINGOFTHINGSANDSTUFF
aws_secret_access_key = TheKeyThatGoesHere1234
region = us-east-1
```

You can set the AWS_CONFIG_FILE environment variable to the path of that file, and the AWS tools will pick them up.

#### Building a Server

To create a server with the current scripts, you must follow these steps:

1. Copy minecraft_specs.json.skel to minecraft_specs.json
2. Run the find_ami script, and copy its output
3. Edit minecraft_specs.json - You'll need to enter information for every field but user_data:
  - placement controls which availability_zone the instance will be created in (Ex: us-east-1a)
  - image_id is where you need to paste the output of the find_ami script (Ex: ami-83e2dcea)
  - key_name is the name of the AWS key pair that you want to use for your server
  - security_group_ids is an array of AWS Security Group IDs that you wish to apply to the new instance
  - iam_instance_profile is the name of the IAM Role that you want the instance to have
  - instance_type is what size of instance you'd like to create (See: http://aws.amazon.com/ec2/pricing/)
  - user_data isn't a field that you can manually supply, as it must be base64 encoded
3. Copy user-data.skel to user-data - This file is the script that runs as the instance starts
  - You'll have to make the config file that is written by this script mirror the config you created during setup, above (aws_access_key_id, aws_secret_access_key, and region)
  - The script currently expects you to have already created a volume that it will try to attach. The ID of this volume must be supplied as the --volume-id argument on the line that begins "aws ec2 attach-volume"
  - The script also expects you to be using an Elastic IP, which it expects to already be provisioned...it'll have to be supplied as the --public-ip argument on the line that begins "aws ec2 associate-address"
  - The script currently forces US/Eastern as the time zone, this can be altered by changing where it mentions "America/New_York". If you delete this line entirely, the instance will run on UTC.
  - The script creates a user account named "minecraft", which it associates with /home/minecraft, which it expects to already exist on the volume.
  - The last line of this file runs a startup script that it expects to exist on the volume.
4. Run the update_user_data script. This will replace the placeholder text in minecraft_specs.json with an encoded copy of the user-data file. If you ever update the user-data file, you'll have to re-run update_user_data, or your next instance will not reflect any changes made.
5. Optionally, edit make_minecraft to change the argument of --spot-price. The current incarnation of the script only creates spot instances (I.E., instances that will be terminated if the market price rises above the price you bid, which is set by --spot-price).
6. Run make_minecraft
 
At this point, a JSON structure should be output, which should say something to the effect of "Your Spot request has been submitted for review, and is pending evaluation." As long as your bid is high enough and Amazon has capacity available, your instance request should be fulfilled within a few minutes. If your bid is low, or they don't have the capacity, it could take a very long time.
