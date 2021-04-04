# Run_Analisys.R
Course project (Getting and cleaning data week 4)

#1
if(!exists("./cleanProject")){dir.create("./cleanProject")}
FileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
path<-"./cleanProject/Data.zip"
download.file(FileUrl,path)

#Descomprimr el archivo
unzip(zipfile = "./cleanProject/Data.zip", exdir = "./cleanProject")

#Leer archivos

#Datos de prueba
prueba_x<-read.table("./cleanProject/UCI HAR Dataset/test/X_test.txt")
prueba_y<-read.table("./cleanProject/UCI HAR Dataset/test/Y_test.txt")
prueba_sujeto<-read.table("./cleanProject/UCI HAR Dataset/test/subject_test.txt")

#Datos de entrenamiento
ent_x<-read.table("./cleanProject/UCI HAR Dataset/train/X_train.txt")
ent_y<-read.table("./cleanProject/UCI HAR Dataset/train/y_train.txt")
ent_sujeto<-read.table("./cleanProject/UCI HAR Dataset/train/subject_train.txt")

#Las caracteristicas, actividades y etiquetas
caracteristicas<-read.table("./cleanProject/UCI HAR Dataset/features.txt")
Actividades<-read.table("cleanProject/UCI HAR Dataset/activity_labels.txt")

#Fusion de entrenamiento y prueba
colnames(ent_x)<-caracteristicas[,2]
colnames(ent_y)<-"actividadID"
colnames(ent_sujeto)<-"sujetoID"

colnames(prueba_x)<-caracteristicas[,2]
colnames(prueba_y)<-"actividadID"
colnames(prueba_sujeto)<-"sujetoID"

colnames(Actividades)<-c("actividadID","tipo de actividad")

#unir
entrenamiento<-cbind(ent_y,ent_sujeto,ent_x)
pruebas<-cbind(prueba_y,prueba_sujeto,prueba_x)
datos<-rbind(entrenamiento,pruebas)
colNombres<-colnames(entrenamiento)
colnames(pruebas)<-colNombres #Igualando nombres de las columnas para poder unir
FullData<-rbind(entrenamiento,pruebas)

#2.Extrar solo informacion de la media y desviacion Estandar de los datos
columnas<-colnames(FullData)
media_desv<-(grepl("actividadID",columnas)|
                     grepl("sujetoID",columnas)|
                     grepl("mean...",columnas)|
                     grepl("std...",columnas)
             )
data_pedida<-FullData[,media_desv==T]

#3 configurando por nombre de actividad
nombreActividad<-merge(data_pedida,Actividades,
                       by="actividadID", all.x = T)

#4 Aplicando nombres descriptivos a las variables
columnas<-colnames(nombreActividad)

#Vamos a eliminar caracteres especiales
columnas<-gsub("[\\(\\)-]","",columnas)

columnas<-gsub("^f", "frequencyDomain",columnas)
columnas<-gsub("^t", "timeDomain",columnas)
columnas<-gsub("Acc", "Acelerometer",columnas)
columnas<-gsub("Gyro", "Gyroscope",columnas)
columnas<-gsub("Mag", "Magnitude",columnas)
columnas<-gsub("Freq", "Frecuency",columnas)
columnas<-gsub("mean", "mean",columnas)
columnas<-gsub("std", "standarDesviation",columnas)

colnames(nombreActividad)<-columnas

#5 promedio de cada variable
library(dplyr)
Tidy<-nombreActividad %>% 
        group_by(actividadID,sujetoID) %>% 
        summarise_each(funs(mean))
write.table(Tidy,"tydyDataset.txt",row.names = F)
