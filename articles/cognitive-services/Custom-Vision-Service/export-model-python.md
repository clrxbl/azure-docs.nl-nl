---
title: 'Zelfstudie: TensorFlow-model uitvoeren in Python - Custom Vision Service'
titlesuffix: Azure Cognitive Services
description: Leer hoe u een TensorFlow-model uitvoert in Python.
services: cognitive-services
author: areddish
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 05/17/2018
ms.author: areddish
ms.openlocfilehash: 26427406b045b96f2f3f612e4444b7dc2afcefc6
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: nl-NL
ms.lasthandoff: 10/03/2018
ms.locfileid: "48247309"
---
# <a name="tutorial-run-tensorflow-model-in-python"></a>Zelfstudie: TensorFlow-model uitvoeren in Python

Nadat u [uw TensorFlow-model](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model) hebt geëxporteerd uit de Custom Vision Service, wordt in deze snelstartgids beschreven hoe u dit model lokaal kunt gebruiken voor het classificeren van afbeeldingen.

## <a name="install-required-components"></a>Vereiste onderdelen installeren

### <a name="prerequisites"></a>Vereisten

Als u de zelfstudie wilt gebruiken, moet u het volgende doen:

- Python 2.7of hoger of Python 3.5 of hoger installeren.
- Pip installeren.

U moet ook de volgende pakketten installeren:

```
pip install tensorflow
pip install pillow
pip install numpy
pip install opencv-python
```

## <a name="load-your-model-and-tags"></a>Uw model en labels laden

Het gedownloade ZIP-bestand bevat een model.pb en een labels.txt. Deze bestanden vertegenwoordigen het getrainde model en de classificatielabels. De eerste stap is het laden van het model in uw project.

```Python
import tensorflow as tf
import os

graph_def = tf.GraphDef()
labels = []

# Import the TF graph
with tf.gfile.FastGFile(filename, 'rb') as f:
    graph_def.ParseFromString(f.read())
    tf.import_graph_def(graph_def, name='')

# Create a list of labels.
with open(labels_filename, 'rt') as lf:
    for l in lf:
        labels.append(l.strip())
```

## <a name="prepare-an-image-for-prediction"></a>Een afbeelding voorbereiden voor voorspelling

Er zijn een paar stappen nodig om de afbeelding voor te bereiden, zodat deze de juiste vorm heeft voor voorspelling. Deze stappen bootsen de afbeeldingsbewerking na die wordt uitgevoerd tijdens de training:

### <a name="open-the-file-and-create-an-image-in-the-bgr-color-space"></a>Open het bestand en maak een afbeelding in de BGR-kleurruimte

```Python
from PIL import Image
import numpy as np
import cv2

# Load from a file
imageFile = "<path to your image file>"
image = Image.open(imageFile)

# Update orientation based on EXIF tags, if the file has orientation info.
image = update_orientation(image)

# Convert to OpenCV format
image = convert_to_opencv(image)
```

### <a name="deal-with-images-with-a-dimension-1600"></a>Pas afbeeldingen aan die groter zijn dan 1600

```Python
# If the image has either w or h greater than 1600 we resize it down respecting
# aspect ratio such that the largest dimension is 1600
image = resize_down_to_1600_max_dim(image)
```

### <a name="crop-the-largest-center-square"></a>Snijd het grootste middelste vak bij

```Python
# We next get the largest center square
h, w = image.shape[:2]
min_dim = min(w,h)
max_square_image = crop_center(image, min_dim, min_dim)
```

### <a name="resize-down-to-256x256"></a>Verklein de afbeelding tot 256 x 256

```Python
# Resize that square down to 256x256
augmented_image = resize_to_256_square(max_square_image)
```


### <a name="crop-the-center-for-the-specific-input-size-for-the-model"></a>Snijd het midden bij afhankelijk van de specifieke invoergrootte voor het model

```Python
# The compact models have a network size of 227x227, the model requires this size.
network_input_size = 227

# Crop the center for the specified network_input_Size
augmented_image = crop_center(augmented_image, network_input_size, network_input_size)

```

In de bovenstaande stappen worden de volgende helperfuncties gebruikt:

```Python
def convert_to_opencv(image):
    # RGB -> BGR conversion is performed as well.
    r,g,b = np.array(image).T
    opencv_image = np.array([b,g,r]).transpose()
    return opencv_image

def crop_center(img,cropx,cropy):
    h, w = img.shape[:2]
    startx = w//2-(cropx//2)
    starty = h//2-(cropy//2)
    return img[starty:starty+cropy, startx:startx+cropx]

def resize_down_to_1600_max_dim(image):
    h, w = image.shape[:2]
    if (h < 1600 and w < 1600):
        return image

    new_size = (1600 * w // h, 1600) if (h > w) else (1600, 1600 * h // w)
    return cv2.resize(image, new_size, interpolation = cv2.INTER_LINEAR)

def resize_to_256_square(image):
    h, w = image.shape[:2]
    return cv2.resize(image, (256, 256), interpolation = cv2.INTER_LINEAR)

def update_orientation(image):
    exif_orientation_tag = 0x0112
    if hasattr(image, '_getexif'):
        exif = image._getexif()
        if (exif != None and exif_orientation_tag in exif):
            orientation = exif.get(exif_orientation_tag, 1)
            # orientation is 1 based, shift to zero based and flip/transpose based on 0-based values
            orientation -= 1
            if orientation >= 4:
                image = image.transpose(Image.TRANSPOSE)
            if orientation == 2 or orientation == 3 or orientation == 6 or orientation == 7:
                image = image.transpose(Image.FLIP_TOP_BOTTOM)
            if orientation == 1 or orientation == 2 or orientation == 5 or orientation == 6:
                image = image.transpose(Image.FLIP_LEFT_RIGHT)
    return image
```

## <a name="predict-an-image"></a>Een afbeelding voorspellen

Als de afbeelding is voorbereid als een tensor, kunnen we het langs het model laten lopen voor een voorspelling:

```Python

# These names are part of the model and cannot be changed.
output_layer = 'loss:0'
input_node = 'Placeholder:0'

with tf.Session() as sess:
    prob_tensor = sess.graph.get_tensor_by_name(output_layer)
    predictions, = sess.run(prob_tensor, {input_node: [augmented_image] })
```

## <a name="view-the-results"></a>De resultaten bekijken

De resultaten van het verwerken van de afbeeldingstensor door het model moet vervolgens weer worden gekoppeld aan de labels.

```Python
    # Print the highest probability label
    highest_probability_index = np.argmax(predictions)
    print('Classified as: ' + labels[highest_probability_index])
    print()

    # Or you can print out all of the results mapping labels to probabilities.
    label_index = 0
    for p in predictions[0]:
        truncated_probablity = np.float64(np.round(p,8))
        print (labels[label_index], truncated_probablity)
        label_index += 1
```
## <a name="next-steps"></a>Volgende stappen

U kunt het model ook verpakken in een mobiele toepassing:
* [Geëxporteerd Tensorflow-model gebruiken in een Android-toepassing](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [Geëxporteerd CoreML-model gebruiken in een Swift iOS-toepassing](https://go.microsoft.com/fwlink/?linkid=857726)
* [Geëxporteerd CoreML-model gebruiken in een iOS-toepassing met Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)

