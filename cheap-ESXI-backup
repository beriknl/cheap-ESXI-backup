import time, logging, shutil, subprocess, fnmatch, re, os, argparse
### usage : -i *the full path to the VMDK* -n *the name as in ESXI* -id *the ID as per ESXI* ###


######################################################################
### provide the backup location where the VMDK should be cloned to ###
######################################################################
backup = "/vmfs/volumes/6771a1ed-f6e38d1b/"


### pass the arguments into variables ###
parser = argparse.ArgumentParser(description='Backup program to shut-down and backup VMDKs')
parser.add_argument("-i", "--input", help="Provide the full path to the VMDK to be backup.")
parser.add_argument("-n", "--name", help="Provide the name of the VM.")
parser.add_argument("-id", "--id", help="Provide the VM id. See in the URL from ESXI e.g.")

args = parser.parse_args()

#############################################
###provide proper naming for the variables###
#############################################
id = args.id
vmdk = args.input
vm_name = args.name

####################################
###check if all arguments are set###
####################################
if None in (id, vmdk, vm_name):
    print("Please make sure to provide all neccesary arguments: -id, -i and -n.")
    exit()

###################################
###Check whether the VMDK exists###
###################################
if not os.path.isfile(vmdk):
    print("VMDK file was not found. Please check the location. Exitting.")
    exit()

#################################
###identify machine to capture###
#################################
pattern = "*" + vm_name + "*"


#######################################
###command to run the esxcli command###
#######################################
cmd = 'esxcli vm process list'
p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE)

list=[]

###############################################################################################
###loop throug the output of the subprocess and put them in a tuple to search through later.###
###############################################################################################
for line in p.stdout:                                                                          
    list.append(str(line))                                                                     
p.wait()                                                                                       
                                                                                               
##############################################                                                 
###search the list for the matching VM name###                                                 
##############################################                                                 
matching = fnmatch.filter(list, pattern)                                                       
                                                                                               
########################################################################################################################################
###the first return is assumed to be the only one matching. From this one the world id is extracted in order to shut down the machine###
########################################################################################################################################
                                                                                                                                        
ref_index = list.index(matching[0])                                                                                                     
str = list[ref_index+1]                                                                                                                 
m = re.match(r"b'   World ID: (\d+)\\n'", str)                                                                                          
                                                                                                                                        
############################################################                                                                            
##from the re match the index 1 return the matched result###                                                                            
############################################################                                                                            
world_id = m.group(1)                                                                                                                   
print(world_id)                                                                                                                         
                                                                                                                                        
##################################                                                                                                      
#Make sure the world ID was set###                                                                                                      
##################################                                                                                                      
try:                                                                                                                                    
    world_id                                                                                                                            
except:                                                                                                                                 
    print ("World ID not found for given VM name. Exitting.")                                                                           
    logging.warning("World ID is not set")                                                                                              
    exit()                                                                                                                              
                                                                                                                                        
##################################################################                                                                      
### what is the backup location.                               ###                                                                      
### for the machine, a subfolder will be created #################                                                                      
### define and create the sub-directory in the backup location ###                                                                      
##################################################################     

file = os.path.basename(vmdk)                                                                                                           
dest = backup + args.name + "/"                                                                                                         
t = time.localtime()                                                                                                                    
timestamp = time.strftime('%b-%d-%Y_%H%M', t)                                                                                           
backup_loc = (dest + timestamp)                                                                                                         
                                                                                                                                        
destVMDK = backup_loc + '/' + file                                                                                                      
                                                                                                                                        
###create the backup location, if it doesnt exist.                                                                                      
                                                                                                                                        
os.makedirs(dest, exist_ok=True)                                                                                                        
                                                                                                                                        
###create the specific time_stamped directory.                                                                                          
os.makedirs(backup_loc, exist_ok=True)                                                                                                  
                                                                                                                                        
                                                                                                                                        
###kill the VM softly###                                                                                                                
kill_cmd = "esxcli vm process kill --type=soft --world-id=" + world_id                                                                  
                                                                                                                                        
try:                                                                                                                                    
    kill = subprocess.Popen(kill_cmd, shell=True, stdout=subprocess.PIPE)                                                               
    kill.wait()                                                                                                                         
except:                                                                                                                                 
    print ("Unable to shutdown Virtual Machine")                                                                                        
    logging.warning("Script unable to power down virtual machine")                                                                      
    exit()                                                                                                                              
                                                                                                                                        
time.sleep(30)                                                                                                                          
                                                                                                                                        
try:                                                                                                                                    
    VMDKcp = subprocess.Popen("vmkfstools -i " + vmdk + " " + destVMDK + " -d thin -a buslogic", shell=True, stdout=subprocess.PIPE)    
    VMDKcp.wait()                                                                                                                       
                                                                                                                                        
except:                                                                                                                                 
    print ("Unable to copy the VM")                                                                                                     
    logging.warning("Script unable to copy the VMDK")                                                                                   
    exit()                                                                                                                              
                                                                                                                                        
time.sleep(5)                                                                                                                           
vim_cmd = "vim-cmd vmsvc/power.on " + id                                                                                                
try:                                                                                                                                    
    VIMcmd = subprocess.run(vim_cmd, shell=True)                                                                                        
    print("Started VM again")                                                                                                           
    logging.info("VM started again.")                                                                                                   
except:                                                                                                                                 
    logging.warning("Could not start VM")                                                                                               
    exit()                               
