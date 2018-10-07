#!/usr/bin/env python2

# AWS Management

import optparse
import ConfigParser
import os, time
import boto.ec2

def start():
    try:
        instances = list(conn.start_instances(instance_ids=instance_id))
        
        if len(instances) == 1 and instances[0].id == instance_id:
            time.sleep(2)
            if conn.associate_address(instance_id=instance_id, public_ip=public_ip) is True:
                print("Tinker: 'All systems go!'")
                exit(0)
        else:
            print("Problem starting instance. Exiting...")
            exit(-1)
    except Exception as e:
        print(e)
        exit(-1)

def stop():
    try:
        instances = list(conn.stop_instances(instance_ids=instance_id))

        if len(instances) == 1 and instances[0].id == instance_id:
            print("Tinker: 'So this is wits, end.'")
            exit(0)
        else:
            print("Problem stopping instance. Exiting...")
            exit(-1)
    except Exception as e:
        print(e)
        exit(-1)

def default():
    print('Unrecognised action. Exiting...')
    exit(-1)


if __name__ == '__main__':
    print("Tinker: 'I Tink! Therefore I am!'")
    HOME=os.environ['HOME']

    # Read AWS Configuration
    config = ConfigParser.ConfigParser()
    config.read(HOME+'/.aws/config')
    
    # Build options
    parser = optparse.OptionParser()
    parser.add_option("-a", "--action", help="Action to execute", dest='action')
    parser.add_option("-p", "--profile", help="Action to execute", dest='profile')
    
    # Parse arguments
    (opts, args) = parser.parse_args()
    
    # Check mandatories
    mandatories = ['action', 'profile']
    for m in mandatories:
        if not opts.__dict__[m]:
            print("Missing %s option." % m)
            parser.print_help()
            exit(-1)
        
    # Get information from config
    try:
        instance_id=config.get('config '+opts.profile,'instance_id')
        public_ip=config.get('config '+opts.profile,'public_ip')
        profile=config.get('config '+opts.profile,'profile')
        aws_access_key_id=config.get('profile '+profile,'aws_access_key_id')
        aws_secret_access_key=config.get('profile '+profile,'aws_secret_access_key')
        region=config.get('profile '+profile,'region')
    except Exception as e:
        print(e)
    
    # Printing information
    print(
        "Action %s for:\n  instance_id = %s\n  public_ip = %s\n  profile = %s" % (opts.action.upper(), instance_id, public_ip, profile)
    )
    
    # Creating connection 
    conn = boto.ec2.connect_to_region(region, aws_access_key_id=aws_access_key_id, aws_secret_access_key=aws_secret_access_key)

    # Switch options start/stop
    {
        'start' : start,
        'stop' : stop
    }.get(opts.action, default)()
