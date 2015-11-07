osm_visualization
=================
###North SLO County
![](http://i.imgur.com/zRVZrfU.gif)
###South SLO County
![](http://i.imgur.com/h23wOTS.gif)

### Download the OSM files 
Download the daily improvement files in OSM from: http://planet.openstreetmap.org/replication/day/000/000/ and http://planet.openstreetmap.org/replication/day/000/001/

https://github.com/SLUGIS/osm_visualization/blob/master/data/retrieve-day000
https://github.com/SLUGIS/osm_visualization/blob/master/data/retrieve-day001


Run:

`$ ./retrieve-day000 arg1 arg2` && `$ ./retrieve-day001 arg3 arg4`

Where:

- arg1 is: 11/01/2014 = 780 (start file)
- arg2 is: 06/08/2015 = 999 (end file)...for folder `/000`  
...moving to folder `/001`...
- arg3 is: 06/08/2015 = 000 (start file)
- arg4 is: 11/05/2015 = 149 (end file)

The Script is based in : https://github.com/ericfischer/ebola/blob/master/retrieve-hourly

Example:

`~/osm_visualization/data$ ./retrieve-day 415 780`

### Convert data to Geojson file

we use: https://github.com/SLUGIS/osm_visualization/blob/master/data/get-edits:
modified from: https://github.com/ericfischer/ebola/blob/master/get-mamou-edits

Run: 

`$ ./get-edits file minlat minlon maxlat maxlon > newfile.geojson`
 
Example bounding box from North San Luis Obispo County:

    $minlat = 35.361056;
    $minlon = -120.977325;
    $maxlat = 35.778828;
    $maxlon = -120.331879;

Example bounding box from South San Luis Obispo County:

    $minlat = 34.992316;
    $minlon = -120.901794;
    $maxlat = 35.398566;
    $maxlon = -120.355225;

Process a single file
    
Example: 

`~/osm_visualization/data$ ./get-edits 415.osc.gz 34.89752 -121.34792 35.79522 -119.47262 > slo415.geojson`

It is possible to process all the files:

Run:

`$ ./process_all start_file end_file minlat minlon maxlat maxlon`

Example: 

` ~/osm_visualization/data$ ./process_all 415 780 34.89752 -121.34792 35.79522 -119.47262`

It will take a while depending on the number of files.

If you want to process the file for some especific user use the file, you have to edit the file: https://github.com/SLUGIS/osm_visualization/blob/master/data/get-edits-by-users, 
specifically the line https://github.com/SLUGIS/osm_visualization/blob/master/data/get-edits-by-users#L69 and add more users, then 

Run:

`$ ./get-edits file minlat minlon maxlat maxlon > newfile.geojson`

Example:

`~/osm_visualization/data$ ./get-edits-by-users 415.osc.gz 34.89752 -121.34792 35.79522 -119.47262 > slo415-users.geojson`

Example processing for all users:

`~/osm_visualization/data$ ./process_all_users 415 780 34.89752 -121.34792 35.79522 -119.47262`

### Creating png files

 You need to install [Tilemill](https://www.mapbox.com/tilemill), and  clone [Projectmill](https://github.com/mapbox/projectmill)

`git clone https://github.com/mapbox/projectmill.git`

#### Get a background PNG File:

In this case we used a Mapbox Studio High Contrast image. We use Eric Fischer's project [tile-stitch](https://github.com/ericfischer/tile-stitch), clone to your machine and run.

Then we check the size of the image: in my case it's:  "width":939, "height":748

#### Create a project in Tilemill:

Create a project in Tilemill called sluosm2014, we used a north & south: https://github.com/SLUGIS/osm_visualization/tree/master/tilemill-project/sluosm2014n (e.g., North). This project was stored in the `/usr/share/mapbox/project/` for this particular Ubuntu/Tilemill server install.

#### Configuration in Projectmill

We need to configure this file https://github.com/SLUGIS/osm_visualization/blob/master/make-config-n.py, the exact lines are:

    project={
            "source": "sluosm2014n", #name of projet in Tilemill
            "destination": "sl"+ str(x),
            "format": "png",
            "minzoom": 1,
            "maxzoom": 16,
            "width":939, #width from tile-stitch image
            "height":748, #height from tile-stitch image
            "mml": {
                    "Layer": [
                      {                                        
                                  "Datasource": {
                                  "file": "/home/ubuntu/osm_visualization/data/sl"+str(x)+".geojson"  #directory of files for process
                                }
                  
                      }
                    ],
                    "advanced": {},
                    "name": "line"+ str(x)
               
                  }
        }


Run:

`$ python make-config-n.py arg1 arg2`

where:
arg1: is the number of the first file that was downloaded before.
arg2: is the number of the last file that was downloaded before.

Example:

`~/osm_visualization$ python make-config-n.py 415 780`

and then execute:

`~/projectmill$ ./index.js --mill  --render  -c config.json -f -t /usr/share/tilemill`

After:

That files are created in: /home/ubuntu/Documents/MapBox/export.

### Created a GIF File

Copy and paste the tile-stitch image from tile-stitch to /MapBox/export and rename the file to one incremental digit less than the first file that was created by Tilemill:

The first file is called slo415.png, so the tile-stitch file was named to slo414.png. Then run:

Example:

`~/Documents/MapBox/export$ mogrify -format gif *.png && gifsicle *.gif > anim.gif`
`~/Documents/MapBox/export$ gifsicle --loop=0 --colors 256 *.gif > anim.gif`

Source for making a gif: http://www.lcdf.org/gifsicle/man.html
