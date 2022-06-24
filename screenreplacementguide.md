# Parts

## Display Model

The display part used for the TiDAL 2022 badge is the `LHS114TM-IG01` by Limito Technology. It also has a [datasheet available here](https://github.com/emfcamp/TiDAL-Hardware/blob/main/datasheets/ER-TFT1.14-1_Datasheet.pdf).

## Sourcing / Purchasing

The display part should be available to purchase here:
- https://buydisplay.com/1-14-inch-tft-lcd-display-ips-panel-screen-135x240-for-smart-watch (Primary source, for bulk orders)
- https://ebay.co.uk/itm/293392674769 (Should be the same product)

# Instructions

0. Use a thin object to get between the gluedot and the display from the non-cable side. Do not pry or pull, gently wiggle the screen away from the gluedot.
1. Remove all traces of the gluedot that was holding the screen in place and disconnect the battery
2. Apply some flux to the existing flex connection
3. Use a soldering iron, and move it back and forth across the connections, while *gently* pulling on the screen. The flex will peel off the PCB.
4. Apply some flux to the pads. If they have bridged together, run the soldering iron across them to separate them, then apply more flux. Do not add more solder.
5. Align the new display flex to the PCB. Use the circles between the connector and the treasure chest for alignment. The triangle next to the flex connector points to pin 1.
6. (optional) use some polyimide/kapton tape to hold the display flex down.
7. Press the soldering iron down on the *middle* of the flex connector and slide it out towards one side, then do the other side. If you don't start from the middle, it will misalign itself. You will see the solder flow through the vias on the fpc connector.
8. Clean off the flux residue with alcohol
9. Use double-sided tape to attach the screen to your badge. 
