---
layout: post
title:  "Gstreamer qtdemux 함수 정리"
subtitle:   "Gstreamer qtdemux 함수 정리"
categories: devlog
tags: gst
---

Gstreamer good plugin 중 하나인  
Gstreamer qtdemux.c 내부의 함수를 정리하기 위한 페이지.  




------------------
------------------
1  
static void gst_qtdemux_class_init (GstQTDemuxClass * klass)  

2  
static void gst_qtdemux_init (GstQTDemux * qtdemux)  

3  
static void gst_qtdemux_dispose (GObject * object)

4  
static void gst_qtdemux_post_no_playable_stream_error (GstQTDemux * qtdemux)


5  
static GstBuffer * _gst_buffer_new_wrapped (gpointer mem, gsize size, GFreeFunc free_func)


6  
static GstFlowReturn gst_qtdemux_pull_atom (GstQTDemux *qtdemux, guint64 offset, guint64 size, GstBuffer ** buf)


7  
static gboolean gst_qtdemux_src_convert (GstQTDemux * qtdemux, GstPad * pad, GstFormat src_format, gint64 src_value, GstFormat dest_format, gint64 * dest_value)


8  
static gboolean gst_qtdemux_get_duration (GstQTDemux * gtdemux, GstClockTime * duration)


9  
static gboolean gst_qtdemux_get_duration (GstQTDemux * qtdemux, GstClockTime * duration)


10  
static gboolean gst_qtdemux_handle_src_query (GstPad * pad, GstObject * parent, GstQuery * query)


11  
static void gst_qtdemux_push_tags (GstQTDemux * qtdemux, QtDemuxStream * stream)

 
12  
static void gst_qtdemux_push_event (GstQTDemux * qtdemux, GstEvent * event)


13  
static gint find_func (QtDemuxSample * s1, gint64 * media_time, gpointer user_data)


14  
static guint32 gst_qtdemux_find_index (GstQTDemux * qtdemux, QtDemuxStream * str, guint64 media_time)


15  
static guint32 
gst_qtdemux_find_index_for_given_media_offset_linear (GstQTDemux * qtdemux,
    QtDemuxStream * str, gint64 media_offset)

16  
static guint32
 gst_qtdemux_find_index_linear (GstQTDemux * qtdemux, QtDemuxStream * str,
    GstClockTime media_time)

17
static guint32 
gst_qtdemux_find_keyframe (GstQTDemux * qtdemux, QtDemuxStream * str,
    guint32 index, gboolean next)

18  
static guint32
 gst_qtdemux_find_segment (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstClockTime time_position)

19  
static void
gst_qtdemux_move_stream (GstQTDemux * qtdemux, QtDemuxStream * str,
    guint32 index)
 
20  
static void 
gst_qtdemux_adjust_seek (GstQTDemux * qtdemux, gint64 desired_time,
    gboolean use_sparse, gboolean next, gint64 * key_time, gint64 * key_offset)

21  
static gboolean
 gst_qtdemux_convert_seek (GstPad * pad, GstFormat * format,
    GstSeekType cur_type, gint64 * cur, GstSeekType stop_type, gint64 * stop)

22  
static gboolean 
gst_qtdemux_do_push_seek (GstQTDemux * qtdemux, GstPad * pad, GstEvent * event)

23  
static gboolean 
gst_qtdemux_perform_seek (GstQTDemux * qtdemux, GstSegment * segment,
    guint32 seqnum, GstSeekFlags flags)

24  
static gboolean 
gst_qtdemux_do_seek (GstQTDemux * qtdemux, GstPad * pad, GstEvent * event)

25  
static gboolean 
qtdemux_ensure_index (GstQTDemux * qtdemux)

26  
static gboolean
 gst_qtdemux_handle_src_event (GstPad * pad, GstObject * parent,
    GstEvent * event)

27  
static void
 gst_qtdemux_find_sample (GstQTDemux * qtdemux, gint64 byte_pos, gboolean fw,
    gboolean set, QtDemuxStream ** _stream, gint * _index, gint64 * _time)

28  
static gchar *
_get_upstream_id (GstQTDemux * demux)

29  
static QtDemuxStream *
_create_stream (GstQTDemux * demux, guint32 track_id)

30  
static gboolean
 gst_qtdemux_setcaps (GstQTDemux * demux, GstCaps * caps)

31  
static void 
gst_qtdemux_reset (GstQTDemux * qtdemux, gboolean hard)

32  
static void
 gst_qtdemux_map_and_push_segments (GstQTDemux * qtdemux, GstSegment * segment)

33  
static void
 gst_qtdemux_stream_concat (GstQTDemux * qtdemux, GPtrArray * dest,
    GPtrArray * src)

34  
static gboolean 
gst_qtdemux_handle_sink_event (GstPad * sinkpad, GstObject * parent,
    GstEvent * event)

35  
static gboolean 
gst_qtdemux_handle_sink_query (GstPad * pad, GstObject * parent,
    GstQuery * query)

36  
static void 
gst_qtdemux_set_index (GstElement * element, GstIndex * index)

37  
static GstIndex *
gst_qtdemux_get_index (GstElement * element)

38  
static void
 gst_qtdemux_stbl_free (QtDemuxStream * stream)

39  
static void 
gst_qtdemux_stream_flush_segments_data (QtDemuxStream * stream)

40  
static void 
gst_qtdemux_stream_flush_samples_data (QtDemuxStream * stream)

41  
static void 
gst_qtdemux_stream_clear (QtDemuxStream * stream)

42  
static void 
gst_qtdemux_stream_reset (QtDemuxStream * stream)

43  
static QtDemuxStream *
gst_qtdemux_stream_ref (QtDemuxStream * stream)

44  
static void 
gst_qtdemux_stream_unref (QtDemuxStream * stream)

45  
static GstStateChangeReturn 
gst_qtdemux_change_state (GstElement * element, GstStateChange transition)

46  
static void
 gst_qtdemux_set_context (GstElement * element, GstContext * context)

47  
static void 
qtdemux_parse_ftyp (GstQTDemux * qtdemux, const guint8 * buffer, gint length)

48  
static void
 qtdemux_handle_xmp_taglist (GstQTDemux * qtdemux, GstTagList * taglist,
    GstTagList * xmptaglist)

49  
static void
 qtdemux_update_default_sample_encryption_settings (GstQTDemux * qtdemux,
    QtDemuxCencSampleSetInfo * info, guint32 is_encrypted, guint8 iv_size,
    const guint8 * kid)

50  
static gboolean
 qtdemux_update_default_piff_encryption_settings (GstQTDemux * qtdemux,
    QtDemuxCencSampleSetInfo * info, GstByteReader * br)

51  
static void
 qtdemux_parse_piff (GstQTDemux * qtdemux, const guint8 * buffer, gint length,
    guint offset)

52  
static void
 qtdemux_parse_uuid (GstQTDemux * qtdemux, const guint8 * buffer, gint length)

53  
static void 
qtdemux_parse_sidx (GstQTDemux * qtdemux, const guint8 * buffer, gint length)

54  
static void 
extract_initial_length_and_fourcc (const guint8 * data, guint size,
    guint64 * plength, guint32 * pfourcc)

55  
static gboolean 
qtdemux_parse_mehd (GstQTDemux * qtdemux, GstByteReader * br)

56  
static gboolean
 qtdemux_parse_trex (GstQTDemux * qtdemux, QtDemuxStream * stream,
    guint32 * ds_duration, guint32 * ds_size, guint32 * ds_flags)

57  
static void 
check_update_duration (GstQTDemux * qtdemux, GstClockTime duration)

58  
static gboolean
 qtdemux_parse_trun (GstQTDemux * qtdemux, GstByteReader * trun,
    QtDemuxStream * stream, guint32 d_sample_duration, guint32 d_sample_size,
    guint32 d_sample_flags, gint64 moof_offset, gint64 moof_length,
    gint64 * base_offset, gint64 * running_offset, gint64 decode_ts,
    gboolean has_tfdt)

59  
static inline QtDemuxStream *
qtdemux_find_stream (GstQTDemux * qtdemux, guint32 id)

60  
static gboolean
qtdemux_parse_mfhd (GstQTDemux * qtdemux, GstByteReader * mfhd,
    guint32 * fragment_number)

61  
static gboolean
 qtdemux_parse_tfhd (GstQTDemux * qtdemux, GstByteReader * tfhd,
    QtDemuxStream ** stream, guint32 * default_sample_duration,
    guint32 * default_sample_size, guint32 * default_sample_flags,
    gint64 * base_offset)

62  
static gboolean 
qtdemux_parse_tfdt (GstQTDemux * qtdemux, GstByteReader * br,
    guint64 * decode_time)

63  
static GstStructure *
qtdemux_get_cenc_sample_properties (GstQTDemux * qtdemux,
    QtDemuxStream * stream, guint sample_index)

64  
static guint8 *
qtdemux_parse_saiz (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstByteReader * br, guint32 * sample_count)

65  
static gboolean 
qtdemux_parse_saio (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstByteReader * br, guint32 * info_type, guint32 * info_type_parameter,
    guint64 * offset)

66  
static void 
qtdemux_gst_structure_free (GstStructure * gststructure)

67  
static gboolean 
qtdemux_parse_cenc_aux_info (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstByteReader * br, guint8 * info_sizes, guint32 sample_count)

68  
static gchar *
qtdemux_uuid_bytes_to_string (gconstpointer uuid_bytes)

69  
static gboolean 
qtdemux_parse_pssh (GstQTDemux * qtdemux, GNode * node)

70  
static gboolean 
qtdemux_parse_moof (GstQTDemux * qtdemux, const guint8 * buffer, guint length,
    guint64 moof_offset, QtDemuxStream * stream)

71  
static gboolean
 qtdemux_parse_tfra (GstQTDemux * qtdemux, GNode * tfra_node)

72  
static gboolean 
qtdemux_pull_mfro_mfra (GstQTDemux * qtdemux)

73  
static guint64 add_offset (guint64 offset, guint64 advance)

74  
static GstFlowReturn
 gst_qtdemux_loop_state_header (GstQTDemux * qtdemux)

75  
static GstFlowReturn
 gst_qtdemux_seek_to_previous_keyframe (GstQTDemux * qtdemux)

76  
static void 
gst_qtdemux_stream_segment_get_boundaries (GstQTDemux * qtdemux,
    QtDemuxStream * stream, GstClockTime offset,
    GstClockTime * _start, GstClockTime * _stop, GstClockTime * _time)

77  
static gboolean 
gst_qtdemux_stream_update_segment (GstQTDemux * qtdemux, QtDemuxStream * stream,
    gint seg_idx, GstClockTime offset, GstClockTime * _start,
    GstClockTime * _stop)

78  
static gboolean 
gst_qtdemux_activate_segment (GstQTDemux * qtdemux, QtDemuxStream * stream,
    guint32 seg_idx, GstClockTime offset)

79  
static gboolean 
gst_qtdemux_prepare_current_sample (GstQTDemux * qtdemux,
    QtDemuxStream * stream, gboolean * empty, guint64 * offset, guint * size,
    GstClockTime * dts, GstClockTime * pts, GstClockTime * duration,
    gboolean * keyframe)

80  
static void 
gst_qtdemux_advance_sample (GstQTDemux * qtdemux, QtDemuxStream * stream)

81
static void
 gst_qtdemux_sync_streams (GstQTDemux * demux)

82  
static GstFlowReturn
 gst_qtdemux_combine_flows (GstQTDemux * demux, QtDemuxStream * stream,
    GstFlowReturn ret)

83  
static GstBuffer *
gst_qtdemux_clip_buffer (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstBuffer * buf)

84  
static GstBuffer *
gst_qtdemux_align_buffer (GstQTDemux * demux,
    GstBuffer * buffer, gsize alignment)

85  
static guint8 *
convert_to_s334_1a (const guint8 * ccpair, guint8 ccpair_size, guint field,
    gsize * res)

86  
static guint8 *
extract_cc_from_data (QtDemuxStream * stream, const guint8 * data, gsize size,
    gsize * cclen)

87  
static GstBuffer *
gst_qtdemux_process_buffer (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstBuffer * buf)

88  
static GstFlowReturn gst_qtdemux_push_buffer (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstBuffer * buf)

89  
static GstFlowReturn
 gst_qtdemux_split_and_push_buffer (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstBuffer * buf)

90  
static GstFlowReturn
 gst_qtdemux_decorate_and_push_buffer (GstQTDemux * qtdemux,
    QtDemuxStream * stream, GstBuffer * buf,
    GstClockTime dts, GstClockTime pts, GstClockTime duration,
    gboolean keyframe, GstClockTime position, guint64 byte_position)

91  
static const QtDemuxRandomAccessEntry *
gst_qtdemux_stream_seek_fragment (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GstClockTime pos, gboolean after)

92  
static gboolean
gst_qtdemux_do_fragmented_seek (GstQTDemux * qtdemux)

93  
static GstFlowReturn
 gst_qtdemux_loop_state_movie (GstQTDemux * qtdemux)

94  
static void 
gst_qtdemux_loop (GstPad * pad)

95  
static gboolean 
has_next_entry (GstQTDemux * demux)

96  
static guint64 
next_entry_size (GstQTDemux * demux)

97  
static void
 gst_qtdemux_post_progress (GstQTDemux * demux, gint num, gint denom)

98  
static gboolean 
qtdemux_seek_offset (GstQTDemux * demux, guint64 offset)

99  
static void
 gst_qtdemux_check_seekability (GstQTDemux * demux)

100  
static void 
gst_qtdemux_drop_data (GstQTDemux * demux, gint bytes)

101  
static void 
gst_qtdemux_check_send_pending_segment (GstQTDemux * demux)

102  
static void
 gst_qtdemux_send_gap_for_segment (GstQTDemux * demux,
    QtDemuxStream * stream, gint segment_index, GstClockTime pos)

103  
static GstFlowReturn gst_qtdemux_chain (GstPad * sinkpad, GstObject * parent, GstBuffer * inbuf)

104  
static GstFlowReturn
 gst_qtdemux_process_adapter (GstQTDemux * demux, gboolean force)


105  
static gboolean
 qtdemux_sink_activate (GstPad * sinkpad, GstObject * parent)


106  
static gboolean
 qtdemux_sink_activate_mode (GstPad * sinkpad, GstObject * parent,
    GstPadMode mode, gboolean active)


107  
static void *
 qtdemux_inflate (void *z_buffer, guint z_length, guint * length)


108  
static gboolean
 qtdemux_parse_moov (GstQTDemux * qtdemux, const guint8 * buffer, guint length)


109  
static gboolean
 qtdemux_parse_container (GstQTDemux * qtdemux, GNode * node, const guint8 * buf,


110  
static gboolean
 qtdemux_parse_theora_extension (GstQTDemux * qtdemux, QtDemuxStream * stream,


111  
static gboolean
 qtdemux_parse_node (GstQTDemux * qtdemux, GNode * node, const guint8 * buffer,
    guint length)


112  
static GNode *
 qtdemux_tree_get_child_by_type (GNode * node, guint32 fourcc)


113  
static GNode *
 qtdemux_tree_get_child_by_type_full (GNode * node, guint32 fourcc,
    GstByteReader * parser)


114  
static GNode *
 qtdemux_tree_get_child_by_index (GNode * node, guint index)


115  
static GNode * qtdemux_tree_get_sibling_by_type_full (GNode * node, guint32 fourcc,
    GstByteReader * parser)

116  
static GNode * qtdemux_tree_get_sibling_by_type (GNode * node, guint32 fourcc)

117  
static void qtdemux_do_allocation (QtDemuxStream * stream, GstQTDemux * qtdemux)

118  
static gboolean pad_query (const GValue * item, GValue * value, gpointer user_data)

119  
static gboolean gst_qtdemux_run_query (GstElement * element, GstQuery * query,
    GstPadDirection direction)

120  
static void gst_qtdemux_request_protection_context (GstQTDemux * qtdemux,
    QtDemuxStream * stream)

121  
static gboolean gst_qtdemux_configure_protected_caps (GstQTDemux * qtdemux,
    QtDemuxStream * stream)

122  
static gboolean 
gst_qtdemux_guess_framerate (GstQTDemux * qtdemux, QtDemuxStream * stream)

123  
static gboolean 
gst_qtdemux_configure_stream (GstQTDemux * qtdemux, QtDemuxStream * stream)

124  
static void
 gst_qtdemux_stream_check_and_change_stsd_index (GstQTDemux * demux,
    QtDemuxStream * stream)

125  
static gboolean 
gst_qtdemux_add_stream (GstQTDemux * qtdemux,
    QtDemuxStream * stream, GstTagList * list)

126  
static GstFlowReturn
 qtdemux_find_atom (GstQTDemux * qtdemux, guint64 * offset,
    guint64 * length, guint32 fourcc)

127  
static GstFlowReturn
 qtdemux_add_fragmented_samples (GstQTDemux * qtdemux)

128  
static gboolean
 qtdemux_stbl_init (GstQTDemux * qtdemux, QtDemuxStream * stream, GNode * stbl)

129  
static gboolean
 qtdemux_parse_samples (GstQTDemux * qtdemux, QtDemuxStream * stream, guint32 n)

130  
static gboolean 
qtdemux_parse_segments (GstQTDemux * qtdemux, QtDemuxStream * stream,
    GNode * trak)

131  
static void 
qtdemux_parse_svq3_stsd_data (GstQTDemux * qtdemux,
    const guint8 * stsd_entry_data, const guint8 ** gamma, GstBuffer ** seqh)

132  
static gchar *
 qtdemux_get_rtsp_uri_from_hndl (GstQTDemux * qtdemux, GNode * minf)


133  
static guint
 qtdemux_parse_amr_bitrate (GstBuffer * buf, gboolean wb)


134  
static gboolean 
qtdemux_parse_transformation_matrix (GstQTDemux * qtdemux,
    GstByteReader * reader, guint32 * matrix, const gchar * atom)


135  
static void 
qtdemux_inspect_transformation_matrix (GstQTDemux * qtdemux,
    QtDemuxStream * stream, guint32 * matrix, GstTagList ** taglist)


136  
static gboolean
qtdemux_parse_protection_scheme_info (GstQTDemux * qtdemux,
    QtDemuxStream * stream, GNode * container, guint32 * original_fmt)


137  
static gint
 qtdemux_track_id_compare_func (QtDemuxStream ** stream1,
    QtDemuxStream ** stream2)

138  
static gboolean
 qtdemux_parse_trak (GstQTDemux * qtdemux, GNode * trak)

139  
static void
 gst_qtdemux_guess_bitrate (GstQTDemux * qtdemux)

140  
static GstFlowReturn
 qtdemux_prepare_streams (GstQTDemux * qtdemux)

141  
static gboolean
 _stream_equal_func (const QtDemuxStream * stream, const gchar * stream_id)

142  
static gboolean
 qtdemux_is_streams_update (GstQTDemux * qtdemux)

143  
static gboolean 
qtdemux_reuse_and_configure_stream (GstQTDemux * qtdemux,
    QtDemuxStream * oldstream, QtDemuxStream * newstream)

144  
static gboolean
 qtdemux_ptr_array_find_with_equal_func (GPtrArray * haystack,
    gconstpointer needle, GEqualFunc equal_func, guint * index_)

145  
static gboolean
 qtdemux_update_streams (GstQTDemux * qtdemux)

146  
static GstFlowReturn
 qtdemux_expose_streams (GstQTDemux * qtdemux)

147  
static inline gboolean 
qtdemux_is_brand_3gp (GstQTDemux * qtdemux, gboolean major)

148  
static inline gboolean
 qtdemux_is_string_tag_3gp (GstQTDemux * qtdemux, guint32 fourcc)

149  
static void
 qtdemux_tag_add_location (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

150   
static void
 qtdemux_tag_add_year (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

151  
static void 
qtdemux_tag_add_classification (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

152  
static gboolean 
qtdemux_tag_add_str_full (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

153  
static void
 qtdemux_tag_add_str (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

154  
static void 
qtdemux_tag_add_keywords (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

155  
static void
 qtdemux_tag_add_num (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag1, const char *tag2, GNode * node)

156  
static void
 qtdemux_tag_add_tmpo (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag1, const char *dummy, GNode * node)
 
157  
static void
 qtdemux_tag_add_uint32 (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag1, const char *dummy, GNode * node)

158  
static void 
qtdemux_tag_add_covr (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag1, const char *dummy, GNode * node)

159  
static void
 qtdemux_tag_add_date (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

160  
static void
 qtdemux_tag_add_gnre (GstQTDemux * qtdemux, GstTagList * taglist,
    const char *tag, const char *dummy, GNode * node)

161  
static void
 qtdemux_add_double_tag_from_str (GstQTDemux * demux, GstTagList * taglist,
    const gchar * tag, guint8 * data, guint32 datasize)

162  
static void 
qtdemux_tag_add_revdns (GstQTDemux * demux, GstTagList * taglist,
    const char *tag, const char *tag_bis, GNode * node)

163  
static void
 qtdemux_tag_add_id32 (GstQTDemux * demux, GstTagList * taglist, const char *tag,
    const char *tag_bis, GNode * node)

164  
static void 
qtdemux_tag_add_blob (GNode * node, GstQtDemuxTagList * qtdemuxtaglist)

165  
static void
 qtdemux_parse_udta (GstQTDemux * qtdemux, GstTagList * taglist, GNode * udta)

166  
static gint
 qtdemux_redirects_sort_func (gconstpointer a, gconstpointer b)

167  
static void
 qtdemux_process_redirects (GstQTDemux * qtdemux, GList * references)
 
168  
static gboolean
 qtdemux_parse_redirects (GstQTDemux * qtdemux)

169  
static GstTagList *
qtdemux_add_container_format (GstQTDemux * qtdemux, GstTagList * tags)

170  
static gboolean 
qtdemux_parse_tree (GstQTDemux * qtdemux)

171  
static int
 read_descr_size (guint8 * ptr, guint8 * end, guint8 ** end_out)

172  
static GList * 
parse_xiph_stream_headers (GstQTDemux * qtdemux, gpointer codec_data,
    gsize codec_data_size)

173  
static void
 gst_qtdemux_handle_esds (GstQTDemux * qtdemux, QtDemuxStream * stream,
    QtDemuxStreamStsdEntry * entry, GNode * esds, GstTagList * list)

174  
static inline GstCaps *
_get_unknown_codec_name (const gchar * type, guint32 fourcc)

175  
static GstCaps * 
qtdemux_video_caps (GstQTDemux * qtdemux, QtDemuxStream * stream,
    QtDemuxStreamStsdEntry * entry, guint32 fourcc,
    const guint8 * stsd_entry_data, gchar ** codec_name)

176  
static guint 
round_up_pow2 (guint n)

177  
static GstCaps *
 qtdemux_audio_caps (GstQTDemux * qtdemux, QtDemuxStream * stream,
    QtDemuxStreamStsdEntry * entry, guint32 fourcc, const guint8 * data,
    int len, gchar ** codec_name)

178  
static GstCaps * 
qtdemux_sub_caps (GstQTDemux * qtdemux, QtDemuxStream * stream,
    QtDemuxStreamStsdEntry * entry, guint32 fourcc,
    const guint8 * stsd_entry_data, gchar ** codec_name)

179  
static GstCaps *
 qtdemux_generic_caps (GstQTDemux * qtdemux, QtDemuxStream * stream,
    QtDemuxStreamStsdEntry * entry, guint32 fourcc,
    const guint8 * stsd_entry_data, gchar ** codec_name)

180  
static void 
gst_qtdemux_append_protection_system_id (GstQTDemux * qtdemux,
    const gchar * system_id)


181  
static const gchar *
qt_demux_state_string (enum QtDemuxState state)



