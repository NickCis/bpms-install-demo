# Use jbossdemocentral/developer as the base
FROM jbossdemocentral/developer

# Maintainer details
MAINTAINER Andrew Block <andy.block@gmail.com>, Eric D. Schabell <erics@redhat.com>

# Environment Variables
ENV BPMS_HOME /opt/jboss/bpms/jboss-eap-7.0
ENV BPMS_VERSION_MAJOR 6
ENV BPMS_VERSION_MINOR 4
ENV BPMS_VERSION_MICRO 0
ENV BPMS_VERSION_PATCH GA
ENV BPMS_VERSION_MICRO_PATCH 4

ENV EAP_VERSION_MAJOR 7
ENV EAP_VERSION_MINOR 0
ENV EAP_VERSION_MICRO 0
#ENV EAP_VERSION_PATCH 4

ENV EAP_INSTALLER=jboss-eap-$EAP_VERSION_MAJOR.$EAP_VERSION_MINOR.$EAP_VERSION_MICRO-installer.jar
ENV BPMS_DEPLOYABLE=jboss-bpmsuite-$BPMS_VERSION_MAJOR.$BPMS_VERSION_MINOR.$BPMS_VERSION_MICRO.$BPMS_VERSION_PATCH-deployable-eap7.x.zip
ENV BPMS_PATCH_FOLDER=jboss-bpmsuite-$BPMS_VERSION_MAJOR.$BPMS_VERSION_MINOR.$BPMS_VERSION_MICRO_PATCH-patch
ENV BPMS_PATCH=$BPMS_PATCH_FOLDER.zip

# ADD Installation Files
COPY support/installation-eap support/installation-eap.variables installs/$BPMS_DEPLOYABLE installs/$BPMS_PATCH installs/$EAP_INSTALLER /opt/jboss/

# Update Permissions on Installers
USER root
RUN chown 1000:1000 /opt/jboss/$EAP_INSTALLER /opt/jboss/$BPMS_DEPLOYABLE /opt/jboss/$BPMS_PATCH
USER 1000

# Prepare and run installer and cleanup installation components
RUN sed -i "s:<installpath>.*</installpath>:<installpath>$BPMS_HOME</installpath>:" /opt/jboss/installation-eap \
    && java -jar /opt/jboss/$EAP_INSTALLER  /opt/jboss/installation-eap -variablefile /opt/jboss/installation-eap.variables \
    #&& $BPMS_HOME/bin/jboss-cli.sh --command="patch apply /opt/jboss/jboss-eap-$EAP_VERSION_MAJOR.$EAP_VERSION_MINOR.$EAP_VERSION_PATCH-patch.zip --override-all" \
    && unzip -qo /opt/jboss/$BPMS_DEPLOYABLE  -d $BPMS_HOME/.. \
    && unzip -q /opt/jboss/$BPMS_PATCH -d $BPMS_HOME/.. \
    && cd $BPMS_HOME/../$BPMS_PATCH_FOLDER \
    && ./apply-updates.sh $BPMS_HOME eap7.x \
    && cd - &> /dev/null \
    && rm -rf /opt/jboss/$BPMS_DEPLOYABLE /opt/jboss/$BPMS_PATCH /opt/jboss/$EAP_INSTALLER /opt/jboss/installation-eap /opt/jboss/installation-eap.variables $BPMS_HOME/standalone/configuration/standalone_xml_history/ $BPMS_HOME/../$BPMS_PATCH_FOLDER

# Create users and add support files
RUN $BPMS_HOME/bin/add-user.sh -a -r ApplicationRealm -u bpmsAdmin -p bpmsuite1! -ro analyst,admin,manager,user,kie-server,kiemgmt,rest-all --silent \
  && $BPMS_HOME/bin/add-user.sh -a -r ApplicationRealm -u erics -p bpmsuite1! -ro analyst,admin,manager,user,kie-server,kiemgmt,rest-all --silent
COPY support/standalone.xml $BPMS_HOME/standalone/configuration/
COPY support/userinfo.properties $BPMS_HOME/standalone/deployments/business-central.war/WEB-INF/classes/

# Swtich back to root user to perform cleanup
USER root

# Fix permissions on support files
RUN chown -R 1000:1000 $BPMS_HOME/standalone/configuration/standalone.xml $BPMS_HOME/standalone/deployments/business-central.war/WEB-INF/classes/userinfo.properties

# Run as JBoss
USER 1000

# Expose Ports
EXPOSE 9990 9999 8080

# Run BPMS
CMD ["/opt/jboss/bpms/jboss-eap-7.0/bin/standalone.sh","-c","standalone.xml","-b", "0.0.0.0","-bmanagement","0.0.0.0"]
