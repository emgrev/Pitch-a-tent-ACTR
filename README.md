
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
    def find_spot(self):                        #self=Motor, parent=MyAgent, parent of parent=Forest
        yield 10
        print "this area seems flat and clear"

    def clear(self):
        yield 5
        print "remove any debris in the way"
        self.parent.parent.debris.location='in_forest'

    def lay_footprint(self):
        yield 2
        print "lay the footprint on the ground"
        self.parent.parent.footprint.location='in_clearing'

    def lay_tent(self):
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

class MyAgent(ACTR):
    focus=Buffer()
    motor=MotorModule()

    def init():
        focus.set('find spot')

    def find_spot(focus='find spot'):                           # if focus buffer has this chunk...
        print ("I have found a good clearing")                  # print
        focus.set('clear spot')                                 # change the focus buffer
        motor.find_spot()

    def clear_area(focus='clear spot'):                         # the rest of the productions are the same 
        print ("there is some debris in the clearing")          # but carry out different actions
        focus.set('lay footprint')
        motor.clear()

    def lay_footprint(focus='lay footprint', debris='location:in_forest'):
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

    def add_pegs(focus='pegs', fly='location:on_tent'):          # wait for the production to complete before stopping the agent
        print ("peg down the corners first, make sure the base is taut")
        focus.set('stop')
        motor.tent_pegs()

    def stop_production(focus='stop', pegs='location:in_ground'):
        self.stop()                                              # stop the agent


Chia=MyAgent()             # name the agent
env=forest()               # name the environment
env.agent=Chia             # put the agent in the environment
ccm.log_everything(env)    # print out what happens in the environment

env.run()                  # run the environment
ccm.finished               # stop the environment

