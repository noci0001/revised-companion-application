# Companion Application Blueprint

The **companion application** is an example to
showcase how to create a vehicle application which senses and actuates signals in the vehicle
for Eclipse Leda with help of Eclipse Velocitas and Eclipse Kuksa
. The aim is not to build the best available application possible but to show how one can use the applied technologies
to build a companion app for the interaction with a vehicle, e.g., to move a seat.
If you are new to the concepts around Eclipse SDV and the mentioned projects
we recommend to read the [SDV Tutorial](https://eclipse-leda.github.io/leda/docs/general-usage/sdv-introduction/) first.

## Description

The idea of the specific companion application introduced in this blueprint is to have a custom application to move the driver seat to positions defined
in a driver profile hosted by a third-party web service.

![Leda Seat Adjuster Use Case](./img/seatadjuster.png)

The setup contains the following components:

- Cloud or mobile trigger: not part of the Leda image, but we simulate it by issuing MQTT messages
- **Seat Adjuster** : Developed with Eclipse Velocitas to be deployed by user
- Eclipse Kuksa.val - **KUKSA Databroker** (pre-installed with Eclipse Leda)
- **Mock Service**: Example provider for Eclipse Kuksa.VAL which mocks the behavior of the vehicle.

In the following paragraphs, we introduce the [architecture and the assumed data flow](https://sdv-blueprints.eclipse.dev/docs/companion-application/architecture-seat-adjuster)
before we explain the [development](https://sdv-blueprints.eclipse.dev/docs/companion-application/develop-seat-adjuster) and [deployment](https://sdv-blueprints.eclipse.dev/docs/companion-application/deploy-seat-adjuster) steps.
If you are more interested in the general development steps, you may directly jump to the [develop seat adjuster](https://sdv-blueprints.eclipse.dev/docs/companion-application/develop-seat-adjuster).
