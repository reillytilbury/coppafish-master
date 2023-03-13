# Notebook Comments
## file_names
*file_names* page contains all files that are used throughout the pipeline. `tile` is the only variable not included in the config file and is worked out automatically.  Page added to *Notebook* automatically as soon as *basic_info* page added.

* **input_dir**: *Directory*.

	Where raw *nd2* files are

* **output_dir**: *Directory*.

	Where notebook is saved

* **tile_dir**: *Directory*.

	Where tile *npy* files saved

* **round**: *List [n_rounds]*.

	Names of *nd2* files for the imaging rounds. If not using, will be an empty list.

* **anchor**: *String or None*.

	Name of *nd2* file for the anchor round. `None` if anchor not used

* **raw_extension**: *String*.

	*.nd2* or *.npy* indicating the data type of the raw data.

* **raw_metadata**: *String or None*.

	If `raw_extension = .npy`, this is the name of the *json* file in `input_dir` which contains the   required metadata extracted from the initial *nd2* files. I.e. it is the output of *coppafish/utils/nd2/save_metadata*

* **dye_camera_laser**: *File*.

	*csv* file giving the approximate raw intensity for each dye with each camera/laser combination

* **code_book**: *File*.

	Text file which contains the codes indicating which dye to expect on each round for each gene

* **scale**: *File*.

	Text file saved containing the `extract['scale']` and `extract['scale_anchor']` values used to create  the tile *npy* files in the *tile_dir*. If the second value is 0, it means `extract['scale_anchor']`  has not been calculated yet. 

	 If the extract step of the pipeline is re-run with `extract['scale']` or  `extract['scale_anchor']` different to values saved here, an error will be raised.

* **spot_details_info**: *File*.

	After each tile is finished in find_spots, information about spots found is saved as array to *npz* file: We save this info as an npz because we're saving a collection of arrays. We have an int16 array [n_spots x 3] containing $y$, $x$, $z$, called spot_details. We have an int16 array called spot_no [n_tiles * n_rounds *  n_channels] which gives number of spots found on that [t,r,c]. We also have an [n_anchor_spots * 1] boolean array named isolated_spots which gives 1 if anchor_spot[s] isolated, 0 o/w

* **psf**: *File or None*.

	*npy* file indicating average spot shape (before padding and scaled to fill *uint16* range). Will be `None` if *2D* pipeline used. File won't exist/used if `config['extract']['deconvolve'] = False`. If *3D*, 1st axis in *npy* file is z.

* **omp_spot_shape**: *File*.

	*npy* file indicating average spot shape in *OMP* coefficient sign images. Saved image is *int8 npy* with only values being -1, 0, 1.

* **omp_spot_info**: *File*.

	After each tile is finished in *OMP*, information about spots found is saved as array to *npy* file: `numpy int16 array [n_spots x 7]` containing $y$, $x$, $z$, `gene_no`, `n_pos_neighb`, `n_neg_neighb`, `tile`. If *3D*, 1st axis in *npy* file is z.

* **omp_spot_coef**: *File*.

	After each tile is finished in *OMP*, coefficients for all spots found is saved as sparse  `csr_matrix` to *npz* file:  

	 `CSR_matrix float [n_spots x n_genes]` 

	 giving coefficient found for each gene for each spot.

* **big_dapi_image**: *File or None*.

	*npz* file of stitched *DAPI* image. None if `nb.basic_info.dapi_channel = None` If *3D*, 1st axis in *npz* file is z.

* **big_anchor_image**: *File*.

	*npz* file of stitched image of `ref_round`/`ref_channel`. Will be stitched anchor if anchor used. If *3D*, 1st axis in *npz* file is z.

* **pciseq**: *List of 2 files*.

	*csv* files where plotting information for *pciSeq* is saved. 

	 `pciseq[0]` is the path where the *OMP* method output will be saved. 

	 `pciseq[1]` is the path where the *ref_spots* method output will be saved. 

	 If files don't exist, they will be created when the function *coppafish/export_to_pciseq* is run.

* **tile**: *List of numpy string arrays [n_tiles][(n_rounds + n_extra_rounds) {x n_channels if 3d}]*.

	*2D*: `tile[t][r]` is the *npy* file containing all channels of tile $t$, round $r$. 

	 *3D*: `tile[t][r][c]` is the *npy* file containing all z planes for tile $t$, round $r$, channel $c$

## basic_info
*basic_info* page contains information that is used at all stages of the pipeline.  Page added to *Notebook* in *pipeline/basic_info.py*

* **is_3d**: *Boolean*.

	`True` if *3D* pipeline used, `False` if *2D*

* **anchor_round**: *Integer or None*.

	Index of anchor round (typically the first round after imaging rounds so `anchor_round = n_rounds`). `None` if anchor not used.

* **anchor_channel**: *Integer or None*.

	Channel in anchor round used as reference and to build coordinate system on. Usually channel with most spots. `None` if anchor not used.

* **dapi_channel**: *Integer or None*.

	Channel in anchor round that contains *DAPI* images. `None` if no *DAPI*.

* **ref_round**: *Integer*.

	Round to align all imaging rounds to. Will be anchor if using.

* **ref_channel**: *Integer*.

	Channel in reference round used as reference and to build coordinate system on. Usually channel with most spots. Will be `anchor_channel` if using anchor round

* **use_channels**: *Integer List [n_use_channels]*.

	Channels in imaging rounds to use throughout pipeline.

* **use_rounds**: *Integer List [n_use_rounds]*.

	Imaging rounds to use throughout pipeline.

* **use_z**: *Integer List [nz]*.

	z planes used to make tile *npy* files

* **use_tiles**: *Integer List [n_use_tiles]*.

	Tiles to use throughout pipeline. For an experiment where the tiles are arranged in a $4 \times 3$ ($n_y \times n_x$) grid,  tile indices are indicated as below: 

	 | 2  | 1  | 0  | 

	 | 5  | 4  | 3  | 

	 | 8  | 7  | 6  | 

	 | 11 | 10 | 9  |

* **use_dyes**: *Integer List [n_use_dyes]*.

	Dyes to use when assigning spots to genes.

* **dye_names**: *String List [n_dyes] or None*.

	Names of all dyes so for gene with code $360...$, gene appears with `dye_names[3]` in round $0$, `dye_names[6]` in round $1$, `dye_names[0]` in round $2$ etc. `None` if each channel corresponds to a different dye.

* **channel_camera**: *Integer List [n_channels] or None*.

	`channel_camera[i]` is the wavelength in *nm* of the camera on channel $i$. `None` if `dye_names = None`.

* **channel_laser**: *Integer List [n_channels] or None*.

	`channel_laser[i]` is the wavelength in *nm* of the laser on channel $i$. `None` if `dye_names = None`.

* **tile_pixel_value_shift**: *Integer*.

	This is added onto every tile (except *DAPI*) when it is saved and removed from every tile when loaded. Required so we can have negative pixel values when save to *npy* as *uint16*. 

	 *Typical=15000*

* **n_extra_rounds**: *Integer*.

	Number of non-imaging rounds, typically 1 if using anchor and 0 if not.

* **n_rounds**: *Integer*.

	Number of imaging rounds in the raw data

* **tile_sz**: *Integer*.

	$yx$ dimension of tiles in pixels

* **n_tiles**: *Integer*.

	Number of tiles in the raw data

* **n_channels**: *Integer*.

	Number of channels in the raw data

* **nz**: *Integer*.

	Number of z-planes used to make the *npy* tile images (can be different from number in raw data).

* **n_dyes**: *Integer*.

	Number of dyes used

* **tile_centre**: *Numpy float array [3]*.

	`[y, x, z]` location of tile centre in units of `[yx_pixels, yx_pixels, z_pixels]`. For *2D* pipeline, `tile_centre[2] = 0`

* **tilepos_yx_nd2**: *Numpy integer array [n_tiles x 2]*.

	`tilepos_yx_nd2[i, :]` is the $yx$ position of tile with *fov* index $i$ in the *nd2* file. 

	 Index 0 refers to `YX = [0, 0]` 

	 Index 1 refers to `YX = [0, 1]` if `MaxX > 0`

* **tilepos_yx**: *Numpy integer array [n_tiles x 2]*.

	`tilepos_yx[i, :]` is the $yx$ position of tile with tile directory (*npy* files) index $i$. Equally, `tilepos_yx[use_tiles[i], :]` is $yx$ position of tile `use_tiles[i]`. 

	 Index 0 refers to `YX = [MaxY, MaxX]` 

	 Index 1 refers to `YX = [MaxY, MaxX - 1]` if `MaxX > 0`

* **pixel_size_xy**: *Float*.

	$yx$ pixel size in microns

* **pixel_size_z**: *Float*.

	$z$ pixel size in microns

* **use_anchor**: *Boolean*.

	`True` if anchor round is used, `False` if not.

## extract
The *extract* page contains variables from `extract_and_filter` step which are used later in the  pipeline. `auto_thresh` is used in `find_spots` step. `hist_values` and `hist_counts` are used for  normalisation between channels in the `call_reference_spots` step.  Page added to *Notebook* in *pipeline/extract_run.py*

* **auto_thresh**: *Numpy float array `[n_tiles x (n_rounds + n_extra_rounds) x n_channels]`*.

	`auto_thresh[t, r, c]` is the threshold spot intensity for tile $t$, round $r$, channel $c$ used for spot detection in the `find_spots` step of the pipeline.

* **hist_values**: *Numpy integer array [n_pixel_values]*.

	All possible pixel values in saved *npy* images i.e. length is approx `np.iinfo(np.uint16).max`

* **hist_counts**: *Numpy integer array `[n_pixel_values x n_rounds x n_channels]`*.

	`hist_counts[i, r, c]` is the number of pixels across all tiles in round $r$, channel $c$ which had the value `hist_values[i]`.

## extract_debug
*extract_debug* page stores variables from `extract_and_filter` step which are not needed later in the pipeline but may be useful for debugging purposes.  Page added to *Notebook* in *pipeline/extract_run.py*

* **n_clip_pixels**: *Numpy integer array [n_tiles x (n_rounds + n_extra_rounds) x n_channels]*.

	`n_clip_pixels[t, r, c]` is the number of pixels in the saved *npy* tile for tile $t$, round $r$, channel $c$ which had intensity exceeding max *uint16* value and so had to be clipped.

* **clip_extract_scale**: *Numpy float array [n_tiles x (n_rounds + n_extra_rounds) x n_channels]*.

	`clip_extract_scale[t, r, c]` is the recommended value for extract_scale such that for tile $t$, round $r$, channel $c$ `n_clip_pixels[t, r, c]` would be 0. Only computed for images where `n_clip_pixels[t, r, c] > 0`

* **r1**: *Integer*.

	Filtering is done with a *2D* difference of hanning filter with inner radius `r1` within which it is positive and outer radius `r2` so annulus between `r1` and `r2` is negative. Should be approx radius of spot. 

	 By default this is 0.5 micron converted to yx-pixel units which is typically 3.

* **r2**: *Integer*.

	Filtering is done with a *2D* difference of hanning filter with inner radius `r1` within which it is positive and outer radius `r2` so annulus between `r1` and `r2` is negative. Units are yx-pixels and by default it will be twice r1. 

	 *Typical: 6*

* **r_dapi**: *Integer or None*.

	Filtering for *DAPI* images is a tophat with `r_dapi` radius. Should be approx radius of object of interest. Typically this is 8 micron converted to yx-pixel units which is typically 48. By default, it is `None` meaning *DAPI* not filtered at all and *npy* file not saved.

* **psf**: *Numpy float array [psf_shape[0] x psf_shape[1] x psf_shape[2]] or None (psf_shape is in config file)*.

	Average shape of spot from individual raw spot images normalised so max is 1 and min is 0. `None` if `config['deconvolve'] = False`.

* **psf_intensity_thresh**: *Float*.

	Intensity threshold used to detect spots in raw images which were used to make the psf. None if `config['deconvolve'] = False` or psf provided without spot detection.

* **psf_tiles_used**: *Integer list or None*.

	Tiles where spots for psf calculation came from. `None` if `config['deconvolve'] = False` or psf provided without spot detection.

* **scale**: *Float*.

	Multiplier applied to filtered *nd2* imaging round images before saving as *npy* so full *uint16* occupied.

* **scale_tile**: *Integer or None*.

	Tile of image that scale was found from. `None` if `config['extract']['scale']` provided.

* **scale_channel**: *Integer or None*.

	Channel of image that scale was found from. `None` if `config['extract']['scale']` provided.

* **scale_z**: *Integer or None*.

	z plane of image that scale was found from. `None` if `config['extract']['scale']` provided.

* **scale_anchor**: *Float or None*.

	Multiplier applied to filtered *nd2* anchor round images before saving as *npy* so full *uint16* occupied. `None` if `use_anchor = False`.

* **scale_anchor_tile**: *Integer or None*.

	Tile of image in anchor round/channel that `scale` was found from. `None` if `config['extract']['scale']` provided or `use_anchor = False`.

* **scale_anchor_z**: *Integer or None*.

	z plane of image in anchor round/channel that `scale_anchor` was found from. `None` if `config['extract']['scale_anchor']` provided or `use_anchor = False`.

* **z_info**: *Integer*.

	z plane in *npy* file from which `auto_thresh` and `hist_counts` were calculated. By default, this is the mid plane.

## find_spots
*find_spots* page contains information about spots found on all tiles, rounds and channels.  Page added to *Notebook* in *pipeline/find_spots.py*

* **isolation_thresh**: *Numpy float array [n_tiles]*.

	Spots found on tile $t$, `ref_round`, `ref_channel` are isolated if annular filtered image is below `isolation_thresh[t]` at spot location. 

	 *Typical: 0*

* **spot_no**: *Numpy int32 array [n_tiles x (n_rounds + n_extra_rounds) x n_channels]*.

	`spot_no[t, r, c]` is the number of spots found on tile $t$, round $r$, channel $c$

* **spot_details**: *Numpy int16 array [n_total_spots x 3]*.

	`spot_details[i,:]` is `[y, x, z]` for spot $i$ $y$, $x$ gives the local tile coordinates in yx-pixels. $z$ gives local tile coordinate in z-pixels (0 if *2D*)

* **isolated_spots**: *Boolean Array [n_anchor_spots x 1]*.

	isolated spots[s] returns a 1 if anchor spot s is isolated and 0 o/w

## stitch
*stitch* page contains information about how tiles were stitched together to give global coordinates. Only `tile_origin` is used in later stages of the pipeline. Note that references to *south* in this section should really be *north* and *west* should be *east*.  Page added to *Notebook* in *pipeline/stitch.py*

* **tile_origin**: *Numpy float array [n_tiles x 3]*.

	`tile_origin[t,:]` is the bottom left $yxz$ coordinate of tile $t$. $yx$ coordinates in yx-pixels and z coordinate in z-pixels.

* **south_start_shift_search**: *Numpy integer array [3 x 3]*.

	Initial search range used to find overlap between south neighbouring tiles `[i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). `[2,:]` is in units of z-pixels and is 0 if *2D*.

* **west_start_shift_search**: *Numpy integer array [3 x 3]*.

	Initial search range used to find overlap between west neighbouring tiles `[i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). `[2,:]` is in units of z-pixels and is 0 if *2D*.

* **south_final_shift_search**: *Numpy integer array [3 x 3]*.

	Final search range used to find overlap between south neighbouring tiles `[i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). `[2,:]` is in units of z-pixels and is 0 if *2D*.

* **west_final_shift_search**: *Numpy integer array [3 x 3]*.

	Final search range used to find overlap between west neighbouring tiles `[i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). `[2,:]` is in units of z-pixels and is 0 if *2D*.

* **south_pairs**: *Numpy integer array [n_south_overlap x 2]*.

	`south_pairs[i, 1]` is the tile to the north of `south_pairs[i, 0]`

* **west_pairs**: *Numpy integer array [n_west_overlap x 2]*.

	`west_pairs[i, 1]` is the tile to the east of `west_pairs[i, 0]`

* **south_shifts**: *Numpy integer array [n_south_overlap x 3]*.

	`south_shifts[i, :]` is the $yxz$ shift found that is applied to `south_pairs[i, 0]` to take it to `south_pairs[i, 1]` 

	 Units: `[yx_pixels, yx_pixels, z_pixels]`, `[:, 2] = 0` if *2D*.

* **west_shifts**: *Numpy integer array [n_west_overlap x 3]*.

	`west_shifts[i, :]` is the $yxz$ shift found that is applied to `west_pairs[i, 0]` to take it to `west_pairs[i, 1]` 

	 Units: `[yx_pixels, yx_pixels, z_pixels]`, `[:, 2] = 0` if *2D*.

* **south_score**: *Numpy float array [n_south_overlap]*.

	`south_score[i]` is approximately the number of matches found for `south_shifts[i, :]`

* **west_score**: *Numpy float array [n_west_overlap]*.

	`west_score[i]` is approximately the number of matches found for `west_shifts[i, :]`

* **south_score_thresh**: *Numpy float array [n_south_overlap]*.

	If `south_score[i]` is below `south_score_thresh[i]`, it indicates `south_shifts[i]` may be incorrect.

* **west_score_thresh**: *Numpy float array [n_west_overlap]*.

	If `west_score[i]` is below `west_score_thresh[i]`, it indicates `west_shifts[i]` found may be incorrect.

* **south_outlier_shifts**: *Numpy integer array [n_south_overlap x 3]*.

	If `south_score[i]` was below `south_score_thresh[i]`, `south_shifts[i]` was found again and old shift recorded as `south_outlier_shifts[i]`. Will be zero if this did not happen.

* **west_outlier_shifts**: *Numpy integer array [n_west_overlap x 3]*.

	If `west_score[i]` was below `west_score_thresh[i]`, `west_shifts[i]` was found again and old shift recorded as `west_outlier_shifts[i]`. Will be zero if this did not happen.

* **south_outlier_score**: *Numpy float array [n_south_overlap]*.

	If `south_score[i]` was below `south_score_thresh[i]`, `south_shifts[i]` was found again and old score recorded as `south_outlier_score[i]`. Will be zero if this did not happen.

* **west_outlier_score**: *Numpy float array [n_west_overlap]*.

	If `west_score[i]` was below `west_score_thresh[i]`, `west_shifts[i]` was found again and old score recorded as `west_outlier_score[i]`. Will be zero if this did not happen.

## register_initial
*register_initial* page contains information about how shift between ref round/channel to each imaging round for each tile was found. These are then used as the starting point for determining the affine transforms. Only `shift` is used in later stages of the pipeline.  Page added to *Notebook* in *pipeline/register_initial.py*

* **shift**: *Numpy integer array [n_tiles x n_rounds x 3]*.

	`shift[t, r, :]` is the $yxz$ shift found that is applied to tile $t$, `ref_round` to take it to tile $t$, round $r$. 

	 Units: `[yx_pixels, yx_pixels, z_pixels]`, `[:, :, 2] = 0` if *2D*. Same as `initial_shift` in *register* page.

* **shift_channel**: *Integer*.

	Channel used to find find shifts between rounds to use as starting point for point cloud registration. Typically this is `ref_channel` or a channel with lots of spots.

* **start_shift_search**: *Numpy integer array [n_rounds x 3 x 3]*.

	`[r, :, :]` is the initial search range used to find shift from reference round to round $r$ for all tiles 

	 `[r, i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). 

	 `[r, 2,:]` is in units of z-pixels and is 0 if *2D*.

* **final_shift_search**: *Numpy integer array [n_rounds x 3 x 3]*.

	`[r, :, :]` is the final search range used to find shift from reference round to round $r$ for all tiles 

	 `[r, i, :]` is the `[min, max, step]` of the search in direction $i$ (0 is $y$, 1 is $x$, 2 is $z$). 

	 `[r, 2,:]` is in units of z-pixels and is 0 if *2D*.

* **shift_score**: *Numpy float array [n_tiles x n_rounds]*.

	`shift_score[t, r]` is is approximately the number of matches found for `shift[t,r]`

* **shift_score_thresh**: *Numpy float array [n_tiles x n_rounds]*.

	If `shift_score[t, r]` is below `shift_score_thresh[t, r]`, it indicates `shift[t,r]` may be incorrect.

* **shift_outlier**: *Numpy integer array [n_tiles x n_rounds x 3]*.

	If `shift_score[t, r]` was below `shift_score_thresh[t, r]`, `shift[t, r]` was found again and old shift recorded as `shift_outlier[t, r]`. Will be zero if this did not happen.

* **shift_score_outlier**: *Numpy float array [n_tiles x n_rounds]*.

	If `shift_score[t, r]` was below `shift_score_thresh[t, r]`, `shift[t, r]` was found again and old score recorded as `shift_score_outlier[t, r]`. Will be zero if this did not happen.

## register
*register* page contains the affine transforms to go from the ref round/channel to each imaging round/channel for every tile. Page added to *Notebook* in *pipeline/register.py*

* **initial_shift**: *Numpy integer array [n_tiles x n_rounds x 3]*.

	`shift[t, r, :]` is the $yxz$ shift found that is applied to tile $t$, `ref_round` to take it to tile $t$, round $r$. Units: `[yx_pixels, yx_pixels, z_pixels]`, `[:, :, 2] = 0` if *2D*. Same as shift in *register_initial* page. DON'T KNOW WHY COPIED THIS - PROBABLY SHOULD REMOVE

* **transform**: *Numpy float array [n_tiles x n_rounds x n_channels x 4 x 3]*.

	`transform[t, r, c]` is the affine transform to get from tile $t$, `ref_round`, `ref_channel` to tile $t$, round $r$, channel $c$ Before applying to coordinates, they must be centered and z coordinates put into units of yx-pixels. If *2D*, z scaling set to 1 while shift and rotation set to 0.

## register_debug
*register_debug* page contains information on how the affine transforms in *register* page were calculated. Page added to *Notebook* in *pipeline/register.py*

* **n_matches**: *Numpy integer array [n_tiles x n_rounds x n_channels]*.

	Number of matches found for each transform. A match is when distance between points is less than `config['register']['neighb_dist_thresh']`.

* **n_matches_thresh**: *Numpy integer array [n_tiles x n_rounds x n_channels]*.

	`n_matches[t, r, c]` must exceed `n_matches_thresh[t, r, c]` otherwise `failed[t, r, c] = True` and transform found again using regularisation.

* **error**: *Numpy float array [n_tiles x n_rounds x n_channels]*.

	Average distance between neighbours closer than `config['register']['neighb_dist_thresh']` for each transform.

* **failed**: *Numpy boolean array [n_tiles x n_rounds x n_channels]*.

	`failed[t, r, c] = True` if `transform[t, r, c]` had too few matches or was anomalous compared to average. `n_matches_thresh` in this page and `scale_dev_thresh`, `shift_dev_thresh` in config file quantify the required matches / deviation.

* **converged**: *Numpy boolean array [n_tiles x n_rounds x n_channels]*.

	This is `False` for transforms where the *ICP* algorithm reached `config['register']['n_iter']` iterations before transform converged.

* **av_scaling**: *Numpy float array [n_channels x 3]*.

	`av_scaling[c]` is the $yxz$ chromatic aberration scale factor to channel $c$ from the `ref_channel` averaged over all rounds and tiles. Expect the $y$ and $x$ scaling to be the same and all scalings to be approx 1.

* **av_shifts**: *Numpy float array [n_tiles x n_rounds x 3]*.

	`av_shifts[t, r]` is the $yxz$ shift from tile $t$, `ref_round` to tile $t$, round $r$ averaged over all channels. All three directions are in yx-pixel units.

* **transform_outlier**: *Numpy float array [n_tiles x n_rounds x n_channels x 4 x 3]*.

	`[t, r, c]` is the final transform found for tile $t$, round $r$, channel $c$ without regularisation. Regularisation only used for $t$,$r$,$c$ indicated by failed and so `transform_outlier = 0` for others.

## ref_spots
*ref_spots* page contains gene assignments and info for spots found on reference round.  Page added to *Notebook* in *pipeline/get_reference_spots.py*.  The variables `gene_no`, `score`, `score_diff`, `intensity` will be set to `None` after `get_reference_spots`.  `call_reference_spots` should then be run to give their actual values. This is so if there is an error in  `call_reference_spots`, `get_reference_spots` won't have to be re-run.

* **local_yxz**: *Numpy int16 array [n_spots x 3]*.

	`local_yxz[s]` are the $yxz$ coordinates of spot $s$ found on `tile[s]`, `ref_round`, `ref_channel`. To get `global_yxz`, add `nb.stitch.tile_origin[tile[s]]`.

* **isolated**: *Numpy boolean array [n_spots]*.

	`True` for spots that are well isolated i.e. surroundings have low intensity so no nearby spots.

* **tile**: *Numpy int16 array [n_spots]*.

	Tile each spot was found on.

* **colors**: *Numpy int32 array [n_spots x n_rounds x n_channels]*.

	`[s, r, c]` is the intensity of spot $s$ on round $r$, channel $c$. `-tile_pixel_value_shift` if that round/channel not used otherwise integer.

* **gene_no**: *Numpy int16 array [n_spots]*.

	`gene_no[s]` is the index of the gene assigned to spot $s$.

* **score**: *Numpy float32 array [n_spots]*.

	`score[s]` is the dot product score, $\Delta_{s0g}$, between `colors[s]` and `bled_codes[gene_no[s]]`. Normalisation depends on `config['call_spots']['dot_product_method']`.

* **score_diff**: *Numpy float16 array [n_spots]*.

	`score_diff[s]` is `score[s]` minus the score for the second best gene assignement for spot $s$.

* **intensity**: *Numpy float32 array [n_spots]*.

	$\chi_s = \underset{r}{\mathrm{median}}(\max_c\zeta_{s_{rc}})$ where $\pmb{\zeta}_s=$ `colors[s, r]/color_norm_factor[r]`.

## call_spots
*call_spots* page contains `bleed_matrix` and expected code for each gene. Page added to *Notebook* in *pipeline/call_reference_spots.py*

* **gene_names**: *Numpy string array [n_genes]*.

	Names of all genes in the code book provided.

* **gene_codes**: *Numpy integer array [n_genes x n_rounds]*.

	`gene_codes[g, r]` indicates the dye that should be present for gene $g$ in round $r$.

* **color_norm_factor**: *Numpy float array [n_rounds x n_channels]*.

	Normalisation such that dividing `colors` by `color_norm_factor` equalizes intensity of channels. `config['call_spots']['bleed_matrix_method']` indicates whether normalisation is for rounds and channels or just channels.

* **initial_raw_bleed_matrix**: *Numpy float array [n_rounds x n_channels x n_dyes]*.

	`initial_raw_bleed_matrix[r, c, d]` is the estimate of the raw intensity of dye $d$ in round $r$, channel $c$. All will be nan if separate dye for each channel.

* **initial_bleed_matrix**: *Numpy float array [n_rounds x n_channels x n_dyes]*.

	Starting point for determination of bleed matrix. If separate dye for each channel, `initial_bleed_matrix[r]` will be the identity matrix for each $r$. Otherwise, it will be `initial_raw_bleed_matrix` divided by `color_norm_factor`.

* **bleed_matrix**: *Numpy float array [n_rounds x n_channels x n_dyes]*.

	For a spot, $s$, which should be dye $d$ in round $r$, we expect `color[s, r]/color_norm_factor[r]` to be a constant multiple of `bleed_matrix[r, :, d]`

* **background_codes**: *Numpy float array [n_channels x n_rounds x n_channels]*.

	These are the background codes for which each spot has a `background_coef`. `background_codes[C, r, c] = 1` if `c==C` and 0 otherwise for all rounds $r$. `nan` if $r$/$c$ outside `use_rounds`/`use_channels`.

* **bled_codes**: *Numpy float array [n_genes x n_rounds x n_channels]*.

	`color[s, r]/color_norm_factor[r]` of spot, $s$, corresponding to gene $g$ is expected to be a constant multiple of `bled_codes[g, r]` in round $r$. `nan` if $r$/$c$ outside `use_rounds`/`use_channels` and 0 if `gene_codes[g,r]` outside `use_dyes`. All codes have L2 norm = 1 when summed across all `use_rounds` and `use_channels`.

* **gene_efficiency**: *Numpy float array [n_genes x n_rounds]*.

	`gene_efficiency[g,r]` gives the expected intensity of gene $g$ in round $r$ compared to that expected by the `bleed_matrix`. It is computed based on the average of isolated spot_colors assigned to that gene which exceed `score`, `score_diff` and `intensity` thresholds given in config file. For all $g$, there is an `av_round[g]` such that `gene_efficiency[g, av_round[g]] = 1`. `nan` if $r$ outside `use_rounds` and 1 if `gene_codes[g,r]` outside `use_dyes`.

* **bled_codes_ge**: *Numpy float array [n_genes x n_rounds x n_channels]*.

	`bled_codes` using `gene_efficiency` information i.e. `bled_codes * gene_efficiency`. All codes have L2 norm = 1 when summed across all `use_rounds` and `use_channels`.

* **background_weight_shift**: *Float*.

	Shift to apply to weighting of each background vector to limit boost of weak spots. The weighting of round $r$ for the fitting of the background vector for channel $c$ is `1 / (spot_color[r, c] + background_weight_shift)` so `background_weight_shift` ensures this does not go to infinity for small `spot_color[r, c]`. Typical `spot_color[r, c]` is 1 for intense spot so `background_weight_shift` is small fraction of this.

* **dp_norm_shift**: *Float*.

	When calculating the dot product score, this is the small shift to apply when normalising `spot_colors` to ensure don't divide by zero. Value is for a single round and is multiplied by `sqrt(n_rounds_used)` when computing dot product score. Expected norm of a `spot_color` for a single round is 1 so `dp_norm_shift` is a small fraction of this.

* **abs_intensity_percentile**: *Numpy float array [100] or None*.

	`abs_intensity_percentile[i]` is the i% percentile of absolute `pixel_colors` on `norm_shift_tile`,  `norm_shift_z`. 

	 This is used to compute `nb.omp.initial_intensity_thresh` if not provided.

* **norm_shift_tile**: *Integer*.

	Tile that is used to compute `abs_intensity_percentile` from which `dp_norm_shift`, `background_weight_shift`  and `intensity_thresh` are computed.

* **norm_shift_z**: *Integer*.

	z-plane that is used to compute `abs_intensity_percentile` from which `dp_norm_shift`, `background_weight_shift`  and `intensity_thresh` are computed.

* **gene_efficiency_intensity_thresh**: *Float*.

	`gene_efficiency` is computed from spots with intensity greater than this. By default, it is set to the `config['call_spots']['gene_efficiency_intensity_thresh_percentile']`  percentile of the `intensity` computed for all pixels on the mid z-plane of the most central tile

## omp
*omp* page contains gene assignments and info for  spots located at the local maxima of the gene coefficients returned by *OMP*. Also contains info about `spot_shape` which indicates the expected sign of the *OMP* coefficient in a neighbourhood centered on a spot.  Page added to *Notebook* in *pipeline/call_spots_omp.py*

* **initial_intensity_thresh**: *Float*.

	To save time in `call_spots_omp`, coefficients only found for pixels with `intensity`  of absolute `spot_colors` greater than `initial_intensity_thresh`. This threshold is set to the `config['omp']['initial_intensity_thresh_percentile']` percentile  of the absolute `intensity` of all pixels on the mid z-plane of the central tile (uses `nb.call_spots.abs_intensity_percentile`). It is also clamped between the min and max values given in config file.

* **shape_tile**: *Integer or None*.

	`spot_shape` was found from spots detected on this tile. `None` if `spot_shape` not computed in this experiment.

* **shape_spot_local_yxz**: *Numpy integer array [n_shape_spots x 3] or None*.

	$yxz$ coordinates on `shape_tile`, `ref_round`/`ref_channel` of spots used to compute `spot_shape` `None` if `spot_shape` not computed in this experiment.

* **shape_spot_gene_no**: *Numpy integer array [n_shape_spots] or None*.

	`shape_spot_gene_no[s]` is the gene that the spot at `shape_spot_local_yxz[s]` was assigned to. `None` if `spot_shape` not computed in this experiment.

* **spot_shape_float**: *Numpy float array [shape_max_size[0] x shape_max_size[1] x shape_max_size[2]] or None*.

	Mean of *OMP* coefficient sign in neighbourhood centered on spot. `None` if `spot_shape` not computed in this experiment.

* **initial_pos_neighbour_thresh**: *Integer*.

	Only spots with number of positive coefficient neighbours greater than this are saved to notebook. 

	 Typical = 4 in *2D* and 40 in *3D* (set to 10% of max number by default).

* **spot_shape**: *Numpy integer array [shape_size_y x shape_size_y x shape_size_x]*.

	Expected sign of *OMP* coefficient in neighbourhood centered on spot. 

	 1 means expected positive coefficient. 

	 -1 means expected negative coefficient. 

	 0 means unsure of expected sign.

* **local_yxz**: *Numpy int16 array [n_spots, 3]*.

	`local_yxz[s]` are the $yxz$ coordinates of spot $s$ found on `tile[s]`, `ref_round`, `ref_channel`. To get `global_yxz`, add `nb.stitch.tile_origin[tile[s]]`.

* **tile**: *Numpy int16 array [n_spots]*.

	Tile each spot was found on.

* **colors**: *Numpy int32 array [n_spots x n_rounds x n_channels]*.

	`[s, r, c]` is the intensity of spot $s$ on round $r$, channel $c$. It will be `-tile_pixel_value_shift` if that round/channel not used otherwise integer.

* **gene_no**: *Numpy int16 array [n_spots]*.

	`gene_no[s]` is the index of the gene assigned to spot $s$.

* **n_neighbours_pos**: *Numpy int16 array [n_spots]*.

	Number of positive pixels around each spot in neighbourhood given by `spot_shape==1`. Max is `sum(spot_shape==1)`.

* **n_neighbours_neg**: *Numpy int16 array [n_spots]*.

	Number of negative pixels around each spot in neighbourhood given by `spot_shape==-1`. Max is `sum(spot_shape==-1)`.

* **intensity**: *Numpy float32 array [n_spots]*.

	$\chi_s = \underset{r}{\mathrm{median}}(\max_c\zeta_{s_{rc}})$ where $\pmb{\zeta}_s=$ `colors[s, r]/color_norm_factor[r]`.

## thresholds
*thresholds* page contains quality thresholds which affect which spots plotted and which are exported to *pciSeq*.  Page added to *Notebook* when *utils/pciseq/export_to_pciseq* is run.

* **intensity**: *Float*.

	Final accepted reference and OMP spots require `intensity > thresholds[intensity]`. This is copied from `config[thresholds]` and if not given there, will be set to  `nb.call_spots.gene_efficiency_intensity_thresh`. intensity for a really intense spot is about 1 so intensity_thresh should be less than this.

* **score_ref**: *Float*.

	Final accepted reference spots are those which pass `quality_threshold` which is: 

	 `nb.ref_spots.score > thresholds[score_ref]` and `intensity > thresholds[intensity]`. 

	 This is copied from `config[thresholds]`. Max score is 1 so `score_ref` should be less than this.

* **score_omp**: *Float*.

	Final accepted *OMP* spots are those which pass `quality_threshold` which is: 

	 `score > thresholds[score_omp]` and `intensity > thresholds[intensity]`. 

	 `score` is given by: 

	 `score = (score_omp_multiplier * n_neighbours_pos + n_neighbours_neg) /  (score_omp_multiplier * n_neighbours_pos_max + n_neighbours_neg_max)`. 

	 This is copied from `config[thresholds]`. Max score is 1 so `score_thresh` should be less than this.

* **score_omp_multiplier**: *Float*.

	Final accepted OMP spots are those which pass quality_threshold which is: 

	 `score > thresholds[score_omp]` and `intensity > thresholds[intensity]`. 

	 score is given by: 

	 `score = (score_omp_multiplier * n_neighbours_pos + n_neighbours_neg) /  (score_omp_multiplier * n_neighbours_pos_max + n_neighbours_neg_max)`. 

	 This is copied from `config[thresholds]`.

