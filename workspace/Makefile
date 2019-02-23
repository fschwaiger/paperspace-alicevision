CAM_DB = /opt/aliceVision/share/aliceVision/cameraSensors.db
VOC_TREE = /opt/aliceVision/share/aliceVision/vlfeat_K80L3.SIFT.tree
BASE_DIR ?= /storage
CACHE_DIR ?= $(BASE_DIR)/alicevision
UNZIP_DIR ?= $(BASE_DIR)/alicevision/images
IMAGE_DIR ?= $(BASE_DIR)/alicevision/images
IMAGE_ZIP ?=
ARTIFACTS ?= /artifacts
SCALING ?= 2
N_IMAGES = $(shell ls $(IMAGE_DIR) | wc -l)
N_IMAGES_PER_GROUP = 16
VERBOSE = warning
START := $(shell date +%s)
ELAPSED = $$(expr $$(date +%s) - $(START))
OK = /.success

.PHONY: clean retry all artifacts meshroom
clean:
	rm -rf $(CACHE_DIR)
retry: $(CACHE_DIR)
	find $(CACHE_DIR) -type d -maxdepth 1 -regex ".*/[0-9][0-9]" -exec test -e "{}$(OK)" ';' -prune -exec rm -rf {} \;
all: clean artifacts

artifacts: retry $(ARTIFACTS)/filtered.obj $(ARTIFACTS)/smoothed.obj
$(ARTIFACTS)/mesh.obj: $(CACHE_DIR)/09$(OK)
$(ARTIFACTS)/filtered.obj: $(CACHE_DIR)/10$(OK)
$(ARTIFACTS)/smoothed.obj: $(CACHE_DIR)/11$(OK)
$(ARTIFACTS)/textured.obj: $(CACHE_DIR)/12$(OK)

meshroom: $(CACHE_DIR) $(IMAGE_DIR)
	meshroom_photogrammetry \
		--input $(IMAGE_DIR) \
		--output $(ARTIFACTS) \
		--scale $(SCALING) \
		--cache $(CACHE_DIR)

$(IMAGE_DIR):
	wget -nv "$(IMAGE_ZIP)" -O $(UNZIP_DIR).zip
	unzip $(UNZIP_DIR).zip -d $(UNZIP_DIR)
	rm $(UNZIP_DIR).zip
	rm -rf $(IMAGE_DIR)/__MACOSX

$(CACHE_DIR):
	@echo "{\"chart\": \"progress\", \"axis\": \"time (seconds)\"}"
	mkdir -p $(CACHE_DIR)

S00 = $(CACHE_DIR)/00
$(S00)$(OK): $(CACHE_DIR) $(IMAGE_DIR)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 0}"
	mkdir -p $(S00)
	aliceVision_cameraInit \
		--verboseLevel $(VERBOSE) \
		--sensorDatabase $(CAM_DB) \
		--imageFolder $(IMAGE_DIR) \
		--output $(S00)/cameraInit.sfm
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 0}"
	touch $(00)$(OK)

S01 = $(CACHE_DIR)/01
$(S01)$(OK): $(S00)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 1}"
	mkdir -p $(S01)
	aliceVision_featureExtraction \
		--verboseLevel $(VERBOSE) \
		--describerTypes sift \
		--forceCpuExtraction True \
		--describerPreset normal \
		--rangeStart 0 \
		--rangeSize $(N_IMAGES) \
		--input $(S00)/cameraInit.sfm \
		--output $(S01)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 1}"
	touch $(01)$(OK)

S02 = $(CACHE_DIR)/02
$(S02)$(OK): $(S01)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 2}"
	mkdir -p $(S02)
	aliceVision_imageMatching \
		--verboseLevel $(VERBOSE) \
		--weights "" \
		--nbMatches 50 \
		--minNbImages 200 \
		--maxDescriptors 500 \
		--tree $(VOC_TREE) \
		--input $(S00)/cameraInit.sfm \
		--featuresFolder $(S01) \
		--output $(S02)/imageMatches.txt
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 2}"
	touch $(02)$(OK)

S03 = $(CACHE_DIR)/03
$(S03)$(OK): $(S02)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 3}"
	mkdir -p $(S03)
	aliceVision_featureMatching \
		--verboseLevel $(VERBOSE) \
		--describerTypes sift \
		--maxMatches 0 \
		--maxIteration 2048 \
		--distanceRatio 0.8 \
		--guidedMatching False \
		--exportDebugFiles False \
		--savePutativeMatches False \
		--geometricEstimator acransac \
		--photometricMatchingMethod ANN_L2 \
		--geometricFilterType fundamental_matrix \
		--imagePairsList $(S02)/imageMatches.txt \
		--input $(S00)/cameraInit.sfm \
		--featuresFolders $(S01) \
		--output $(S03)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 3}"
	touch $(03)$(OK)

S04 = $(CACHE_DIR)/04
$(S04)$(OK): $(S03)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 4}"
	mkdir -p $(S04)
	if [ -d $(S04)/localBA ]; then rm -rf $(S04)/localBA; fi
	aliceVision_incrementalSfM \
		--verboseLevel $(VERBOSE) \
		--minNumberOfObservationsForTriangulation 2 \
		--minAngleForTriangulation 3.0 \
		--minAngleForLandmark 2.0 \
		--maxAngleInitialPair 40.0 \
		--maxNumberOfMatches 0 \
		--localizerEstimator acransac \
		--describerTypes sift \
		--lockScenePreviouslyReconstructed False \
		--localBAGraphDistance 1 \
		--initialPairA "" \
		--initialPairB "" \
		--interFileExtension .abc \
		--useLocalBA True \
		--minInputTrackLength 2 \
		--useOnlyMatchesFromInputFolder False \
		--maxReprojectionError 4.0 \
		--minAngleInitialPair 5.0 \
		--input $(S00)/cameraInit.sfm \
		--featuresFolders $(S01) \
		--matchesFolders $(S03) \
		--outputViewsAndPoses $(S04)/cameras.sfm \
		--extraInfoFolder $(S04) \
		--output $(S04)/bundle.sfm
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 4}"
	touch $(04)$(OK)

S05 = $(CACHE_DIR)/05
$(S05)$(OK): $(S04)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 5}"
	mkdir -p $(S05)
	aliceVision_prepareDenseScene \
		--verboseLevel $(VERBOSE) \
		--input $(S04)/bundle.sfm \
		--output $(S05)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 5}"
	touch $(05)$(OK)

S06 = $(CACHE_DIR)/06
$(S06)$(OK): $(S05)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 6}"
	aliceVision_cameraConnection \
		--verboseLevel $(VERBOSE) \
		--ini $(S05)/mvs.ini
	ln -s $(S05) $(S06)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 6}"
	touch $(06)$(OK)

S07 = $(CACHE_DIR)/07
$(S07)$(OK): $(S06)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 7}"
	mkdir -p $(S07)
	for RANGE_START in $(shell seq 0 $(N_IMAGES_PER_GROUP) $(N_IMAGES)); do \
		aliceVision_depthMapEstimation \
			--verboseLevel $(VERBOSE) \
			--downscale $(SCALING) \
			--ini $(S06)/mvs.ini \
			--output $(S07) \
			--rangeSize $(N_IMAGES_PER_GROUP) \
			--rangeStart $$RANGE_START \
			--refineGammaC 15.5 \
			--refineGammaP 8.0 \
			--refineMaxTCams 6 \
			--refineNDepthsToRefine 31 \
			--refineNiters 100 \
			--refineNSamplesHalf 150 \
			--refineSigma 15 \
			--refineUseTcOrRcPixSize False \
			--refineWSH 3 \
			--sgmGammaC 5.5 \
			--sgmGammaP 8.0 \
			--sgmMaxTCams 10 \
			--sgmWSH 4; \
		echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 7}"; \
	done
	touch $(07)$(OK)

S08 = $(CACHE_DIR)/08
$(S08)$(OK): $(S07)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 8}"
	mkdir -p $(S08)
	aliceVision_depthMapFiltering \
		--verboseLevel $(VERBOSE) \
		--minNumOfConsistensCamsWithLowSimilarity 4 \
		--minNumOfConsistensCams 3 \
		--pixSizeBall 0 \
		--pixSizeBallWithLowSimilarity 0 \
		--nNearestCams 10 \
		--ini $(S06)/mvs.ini \
		--output $(S08) \
		--depthMapFolder $(S07)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 8}"
	touch $(08)$(OK)

S09 = $(CACHE_DIR)/09
$(S09)$(OK): $(S08)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 9}"
	mkdir -p $(S09)
	aliceVision_meshing \
		--verboseLevel $(VERBOSE) \
		--maxInputPoints 50000000 \
		--maxPoints 5000000 \
		--maxPointsPerVoxel 1000000 \
		--repartition multiResolution \
		--partitioning singleBlock \
		--minStep 2 \
		--pixSizeMarginInitCoef 2.0 \
		--pixSizeMarginFinalCoef 4.0 \
		--voteMarginFactor 4.0 \
		--contributeMarginFactor 2.0 \
		--simGaussianSizeInit 10.0 \
		--simGaussianSize 10.0 \
		--minAngleThreshold 1 \
		--simFactor 15.0 \
		--angleFactor 15.0 \
		--refineFuse True \
		--ini $(S06)/mvs.ini \
		--depthMapFilterFolder $(S08) \
		--depthMapFolder $(S07) \
		--output $(S09)/mesh.obj
	cp $(S09)/mesh.obj $(ARTIFACTS)/mesh.obj
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 9}"
	touch $(09)$(OK)

S10 = $(CACHE_DIR)/10
$(S10)$(OK): $(S09)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 10}"
	mkdir -p $(S10)
	aliceVision_meshFiltering \
		--verboseLevel $(VERBOSE) \
		--removeLargeTrianglesFactor 60.0 \
		--iterations 5 \
		--keepLargestMeshOnly True \
		--lambda 1.0 \
		--input $(S09)/mesh.obj \
		--output $(S10)/mesh.obj
	cp $(S10)/mesh.obj $(ARTIFACTS)/filtered.obj
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 10}"
	touch $(10)$(OK)

S11 = $(CACHE_DIR)/11
$(S11)$(OK): $(S10)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 11}"
	mkdir -p $(S11)
	aliceVision_meshDenoising \
		--verboseLevel $(VERBOSE) \
		--denoisingIterations 1 \
		--meshUpdateClosenessWeight 0.001 \
		--lambda 2 \
		--eta 1.5 \
		--mu 1.5 \
		--nu 0.3 \
		--meshUpdateMethod 0 \
		--input $(S10)/mesh.obj \
		--output $(S11)/mesh.obj
	cp $(S11)/mesh.obj $(ARTIFACTS)/smoothed.obj
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 11}"
	touch $(11)$(OK)

S12 = $(CACHE_DIR)/12
$(S12)$(OK): $(S09)$(OK)
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 12}"
	mkdir -p $(S12)
	aliceVision_texturing \
		--verboseLevel $(VERBOSE) \
		--textureSide 8192 \
		--downscale 2 \
		--padding 15 \
		--unwrapMethod Basic \
		--outputTextureFileType png \
		--flipNormals False \
		--fillHoles False \
		--inputDenseReconstruction $(S09)/denseReconstruction.bin \
		--inputMesh $(S10)/mesh.obj \
		--ini $(S06)/mvs.ini \
		--output $(S12)
	cp $(S12)/mesh.obj $(ARTIFACTS)/textured.obj
	@echo "{\"chart\": \"progress\", \"x\": $(ELAPSED), \"y\": 12}"
	touch $(12)$(OK)

