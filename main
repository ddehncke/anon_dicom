from os.path import isfile, join, exists
from os import listdir, mkdir
import pandas as pd
from re import sub
import pydicom
import argparse

# important tags are for example tags that are concerning the order of images or the plane the image was taken
important_tags = ['PerformedProcedureStepDescription', 'InstanceNumber',
                  'StudyDescription', 'SeriesDescription', 'SpecificCharacterSet',
                  'ImageType', '', 'Modality', 'Manufacturer', 'ReferencedImageSequence', 'SourceImageSequence', '',
                  'BodyPartExamined', 'SliceThickness', 'KVP', 'DataCollectionDiameter', 'DeviceSerialNumber',
                  'SoftwareVersions', 'ReconstructionDiameter', 'GantryDetectorTilt', 'TableHeight',
                  'RotationDirection', 'ExposureTime', 'XRayTubeCurrent', 'Exposure', 'FilterType', 'GeneratorPower',
                  'FocalSpots', 'DateOfLastCalibration', 'TimeOfLastCalibration', 'ConvolutionKernel',
                  'PatientPosition', 'ImagePositionPatient', 'ImageOrientationPatient', 'FrameOfReferenceUID',
                  'PositionReferenceIndicator', 'SliceLocation', 'SamplesPerPixel', 'PhotometricInterpretation',
                  'Rows', 'Columns', 'PixelSpacing', 'BitsAllocated', 'BitsStored', 'HighBit', 'PixelRepresentation',
                  'SmallestImagePixelValue', 'LargestImagePixelValue', 'WindowCenter', 'WindowWidth',
                  'RescaleIntercept', 'RescaleSlope', 'WindowCenterWidthExplanation', 'PixelData']

def main(source_folder, target_folder, patient_file, log_file):
    target_file = target_folder + '\\' + patient_file
    log_file = target_folder + '\\' + log_file

    if not exists(target_folder):
        mkdir(target_folder)

    if len(listdir(target_folder)) != 0:
        print(target_folder, " is not empty, please provide an empty folder")
        return


    # list of patient folders in the source_folder
    patients= [join(source_folder, d1) for d1 in listdir(source_folder) if not isfile(join(source_folder, d1))]
    file = open(log_file, "a")

    df = pd.DataFrame()
    counter = 0

    # for each patient folder go through all image series
    for p in patients:

        # for every 5th patient, print the current progress
        if counter % 5 == 0 or counter == len(patients):
            print(int(round(counter/len(patients), 2) * 100), '%')

        counter += 1

        # create a new patient folder in the target folder
        new_patient_folder = target_folder + '\\' + str(p).split('\\')[-1]
        if not exists(new_patient_folder):
            mkdir(new_patient_folder)

        patient_folders = [d1 for d1 in listdir(p) if not isfile(join(p, d1))]
        patient_name = ''
        series_date = ''

        # for each image series folder, create a new folder. Read in the images from the source folder, remove the
        # private_tags and all tags that are not in the list important_tags.
        for folder in patient_folders:
            if not exists(new_patient_folder + '\\' + folder):
                mkdir(new_patient_folder + '\\' + folder)

            files = [d1 for d1 in listdir(join(p, folder))]
            save_tags_of_first_file_2_log = True

            # patient_name and series_date are saved to patient_info.csv, in the case that you want to match the folder
            # paths to the fracture data later on.
            for f in files:
                ds = pydicom.dcmread(join(p, folder, f))
                if 'PatientName' in ds:
                    patient_name = ds.data_element('PatientName').value
                if 'SeriesDate' in ds:
                    series_date = ds.data_element('SeriesDate').value

                # removal of tags marked as private
                ds.remove_private_tags()

                for t in ds:
                    if t.keyword not in important_tags:
                        if t.keyword in ds:
                            delattr(ds, t.keyword)

                save_to_file = join(new_patient_folder, folder, f)

                # for each folder, print the remaining tags after the removal of private tags of the first image to
                # logs.txt. This is a additional security feature to verify all critical tags have been deleted.
                if save_tags_of_first_file_2_log:
                    file.write('\n\n\n')
                    file.write(save_to_file+'\n')
                    file.write(str(ds))
                    save_tags_of_first_file_2_log = False

                # save the image. If the saving does not work, print the filename
                try:
                    ds.save_as(save_to_file)
                except:
                    file.write("unable to save file: " + save_to_file)
                    print(save_to_file)

        patient_name = sub(r'[^a-z]', '', str(patient_name).lower())

        df = df.append({"path": p, "name": patient_name, 'series_date': series_date}, ignore_index=True)
    print('100 %')
    df.to_csv(target_file)
    file.close()
    print("Finished anonymization, the anonymized data can be found in ", target_folder)

if __name__ == '__main__':
    # needs the source folder and target folder. In the target folder there will be the anonymised data,
    # a patient information file and a log file. The patient information file contains the names of the
    # patients, the date when the CT was recorded and the folder the anonymised data was saved to.
    # For each folder of each Patient, the tags of the first image after anonymising are save to the log file.

    parser = argparse.ArgumentParser(description='anonymize dicom images')
    parser.add_argument('--source', metavar='path',
                        help='path to source folder', required=True)
    parser.add_argument('--target', metavar='path',
                        help='path to target folder', required=True)

    args = parser.parse_args()
    print("Starting anonymization")
    main(source_folder=args.source, target_folder=args.target, patient_file='patient_info.csv', log_file='log.txt')
