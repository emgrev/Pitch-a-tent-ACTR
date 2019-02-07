##### pitch a tent production model #####
#Emiy Greve# 

# this is a basic ACT-R model
# the model is a basic production system
# the focus buffer represents the focus of thought
# productions fire when the production matches the focus buffer
# once the production fires, the focus buffer changes
# the motor module that allows the agent to act on objects in the environment
# there is a declarative memory thing

import ccm
log=ccm.log(html=True)

from ccm.lib.actr import *

# Build the environment
# items look and act like chunks - but note the syntactic differences

class forest(ccm.Model):
    clearing=ccm.Model(isa='clearing', location='near', feature1='flat', salience=0.5)
    clearing2=ccm.Model(isa='clearing', location='near', feature1='slanted', salience=0.5)
    clearing3=ccm.Model(isa='clearing', location='near', feature1='dead_deer', salience=0.5)   
    footprint=ccm.Model(isa='footprint', colour='green', location='in_bag')    
    tent=ccm.Model(isa='tent', colour='beigh', material='mesh', location='in_bag')
    poles=ccm.Model(isa='tent poles', colour='blue', location='in_bag')
    pegs=ccm.Model(isa='tent peg', colour='black', location='in_bag')
    fly=ccm.Model(isa='tent fly', colour='green', location='in_bag')
    debris=ccm.Model(isa='debris', includes='pinecones, rocks, sticks', location='in_clearing')

# creat a motor module
# the motor module is technically outside the agent
# but is controlled from within

class MotorModule(ccm.Model):

    def clear(self):
        yield 5
        print "remove any debris in the way"
        self.parent.parent.debris.location='in_forest'

    def lay_footprint(self):
        yield 2
        print "lay the footprint on the ground"
        self.parent.parent.footprint.location='in_clearing'

    def lay_tent(self):                                          # self=Motor, parent=MyAgent, parent of parent=Forest
        yield 2
        print "lay the tent over the footprint"
        self.parent.parent.tent.location='on_footprint'

    def tent_poles(self):
        yield 5
        print "assemble the tent poles"
        self.parent.parent.poles.location='in_clearing'

    def add_tent_poles(self):
        yield 5
        print "attatch the poles to the tent clips to make the skeleton"
        self.parent.parent.poles.location='on_tent'

    def fly_cover(self):
        yield 2
        print "cover the tent with the fly, clip the fly into place"
        self.parent.parent.fly.location='on_tent'

    def tent_pegs(self):
        yield 5
        print "secure the tent to the ground with pegs"
        self.parent.parent.pegs.location='in_ground'
        

# create the agent
# declarative memory (DM) is build directly in the agent

class MyAgent(ACTR):
    focus=Buffer()
    motor=MotorModule()

    DMbuffer=Buffer()                                           # create a buffer for declarative memory
    DM=Memory(DMbuffer, latency=0.05,threshold=0)               # create DM and connect it to its buffer
                                                                # latency controls the relationship between activation and recall
                                                                # activation must be above threshold - can be set to none
##    dm_n=DMNoise(DM,noise=0.0,baseNoise=0.0)                  # turn on for DM subsymbolic processing
##    dm_bl=DMBaseLevel(DM,decay=0.5,limit=None)                # turn on for DM subsymbolic processing

    Vbuffer=Buffer()
    Vmodule=SOSVision(Vbuffer,delay=0)                          # delay=0 means the results of the visual search are
                                                                # placed in the visual buffer right after the request
                                                                # but the request takes 50 msec and the retieval takes 50 msec
                                                                # so actually it takes 100 msec to get the results at minimum

    def init():
        DM.add('cue:clearing tool:footprint a:test')            # put a chunk into DM
        DM.add('cue:footprint tool:tent b:test')
        DM.add('cue:tent tool:fly c:test')

        focus.set('find spot')

    def find_spot(focus='find spot'):                           
        Vmodule.request('isa:clearing location:near')        
        focus.set('check spot')                                 
        print ("I am looking for a good clearing") 


    def check_area(focus='check spot', Vbuffer='isa:clearing location:near feature1:?feature1'):   
        print ("I have found a clearing")
        focus.set('check ?feature1')

    def check_yes(focus='check flat'):
        print ("It is a good flat spot")
        focus.set('set_up')
        Vbuffer.clear

    def check_no(focus='check slanted'):
        focus.set('find spot')
        Vbuffer.clear
        print ("It's a good clearing, but slightly slanted")
        
    def check_hell_no(focus='check dead_deer'):
        print ("What the fuck.")
        focus.set('find spot')
        Vbuffer.clear

    def not_found(focus='check spot'):
        focus.set('find spot')
        Vbuffer.clear
        print ("I need to find a good spot")
                                                                
    def Set_up(focus='set_up'):                                                           
        print ("There is debris on the spot that needs to be cleared away.")                           
        print ("What do I need to do next?")
        DM.request('cue:? tool:?tool a:test')                   # retrieve a chunk from DM into the DM buffer
        focus.set('first thing')                                # ? means the a slot can match any content
        motor.clear()

    def remember(focus='first thing', DMbuffer='cue:?clearing tool:?tool', debris='location:in_forest'):
        print ("oh yea, I need to put down the...")             # match to DMbuffer as well
        print tool                                              # take off the question mark
        focus.set('lay footprint')

    def forget(focus='first thing', DMbuffer=None, DM='error:True'):
                                                                # DMbuffer=none means the buffer is empty
                                                                # DM='error:True' means the search was unsucessful
        print "I recall I needed...."
        print "Fuck, I forgot"
        focus.set('stop')
        
    def lay_footprint(focus='lay footprint'):
        print ("The footprint needs to be layed out flat in the clearing")
        focus.set('lay tent')
        motor.lay_footprint()
        
    def lay_tent(focus='lay tent', footprint='location:in_clearing'):
        print ("the tent should be layed ontop of the footprint")
        focus.set('build poles')
        motor.lay_tent()

    def build_poles(focus='build poles', tent='location:on_footprint'):
        print ("The poles should be assembled")
        focus.set('attatch poles')
        motor.tent_poles()

    def add_poles(focus='attatch poles', poles='location:in_clearing'):
        print ("The poles ends should be place in the holes, and clip the tent to the poles")
        focus.set('fly cover')
        motor.add_tent_poles()

    def cover_tent(focus='fly cover', poles='location:on_tent'):
        print ("The fly should cover the tent")
        focus.set('pegs')
        motor.fly_cover()
                                      
        
    def add_pegs(focus='pegs', fly='location:on_tent'):                                                        
        print ("peg down the corners first, make sure the base is taut")
        focus.set('stop')                                       # wait for the production to complete before stopping the agent
        motor.tent_pegs()

    def stop_production(focus='stop', pegs='location:in_ground'):
        self.stop()                                             # stop the agent


Chia=MyAgent()             # name the agent
env=forest()               # name the environment
env.agent=Chia             # put the agent in the environment
ccm.log_everything(env)    # print out what happens in the environment

env.run()                  # run the environment
ccm.finished               # stop the environment


