# BASH_NDVI_calculation
Following are the source code for the calculation of NDVI from the satellite Landsat images



## unzip the files 

for i in *.tar.gz; do tar -xzvf $i; rm $i; done
## convert latlong
for i in *.TIF; do gdalwarp -t_srs "+proj=latlong +datum=wgs84" $i ${i}_latlong.TIF 

## open grass 
grass 
r.in.gdal -e input=sample.TIF output=basefile location=NDVI

## exit grass and enter the new location 
for i in *_latlong.TIF; do r.in.gdal -e input=$i ouput=`echo $i | cut -d . -f1`; rm *.TIF; done

## run atmospheric correction 
for i in *_MTL.txt; do i.landsat.toar input=`echo $i | cut -d _ -f1-7`_B output=`echo $i | cut -d _ -f1-7`.toar  metfile=$i method=dos1 

## run vegetation index 
##store filenames in textfiles 
g.list type=rast pat=*toar5 > toar5.txt 
g.list type=rast pat=*toar4 > toar4.txt 
wc -l toar5.txt 
for i in `seq 1 65`; do nir=`sed -n ${i}p toar5.txt`; red=`sed -n ${i}p toar4.txt`; g.region rast=$red -p ; i.vi red=$red nir=$nir type=ndvi output=${nir}_NDVI; done 

for i in `g.list type=rast pat=*_ndvi`; do echo $i; g.region rast=$i -p; r.out.gdal input=$i output=`echo $i | cut -d _ -f1-7`_NDVI.TIF ; done
