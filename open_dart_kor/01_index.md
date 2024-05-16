# Index Page

| COMPONENT | NAME |
| --------- | ---- |
| URL     | /index.do    |
| Controller | EgovComIndexController.java |
| Method | index() |
| Service | EgovArticleService.java |
| ServiceImpl | EgovArticleServiceImpl.java |
| Method | selectArticleList(BoardVo) |
| Method | selectNoticeArticleList(BoardVo) |
| DAO | EgovArticleDAO.java |
| Method | selectArticleList(BoardVo) |
| Method | selectArticleListCnt(BoardVo) |
| Method | selectNoticeArticleList(BoardVo) |
| Mapper | EgovArticle_SQL_oracle.xml |
  

```
/* selectArticleList */
SELECT
    *
FROM ( SELECT
    ROWNUM rn,
    tb.* FROM ( SELECT
            a.ntt_id,
            a.ntt_sj,
            a.frst_register_id,
            nvl(b.USER_EMAIL, a.ntcr_nm) AS frst_register_nm,
            TO_CHAR(a.frst_regist_pnttm, 'YYYY-MM-DD') AS frst_regist_pnttm,
            a.rdcnt,
            a.parntsctt_no,
            a.answer_at,
            a.answer_lc,
            a.use_at,
            a.atch_file_id,
            a.bbs_id,
            rtrim(a.ntce_bgnde) ntce_bgnde,
            rtrim(a.ntce_endde) ntce_endde,
            a.sj_bold_at,
            a.notice_at,
            a.secret_at,
            c.comment_co,
            CASE
            WHEN A.BBS_ID = 'B0000000000000000001'
            THEN CASE WHEN (TO_DATE(A.NTCE_BGNDE, 'YYYYMMDD') > SYSDATE - 3) THEN 'Y' ELSE 'N' END
            ELSE CASE WHEN (A.FRST_REGIST_PNTTM > SYSDATE - 3) THEN 'Y' ELSE 'N' END
            END AS NEWAT
        FROM
            comtnbbs a
            LEFT OUTER JOIN TBL_OD_USER b ON a.frst_register_id = b.USER_SEQ
            LEFT OUTER JOIN (
                SELECT
                    ntt_id,
                    bbs_id,
                    COUNT(1) AS comment_co
                FROM
                    comtncomment
                WHERE
                    use_at = 'Y'
                GROUP BY
                    ntt_id,
                    bbs_id
            ) c ON a.ntt_id = c.ntt_id
                    AND a.bbs_id = c.bbs_id
        WHERE
            a.BBS_ID = #{bbsId}
    AND a.USE_AT = 'Y'

    <if test="searchCnd == 0">AND
            a.NTT_SJ LIKE '%' || #{searchWrd} || '%' 		
    </if>
    <if test="searchCnd == 1">AND
            a.NTT_CN LIKE '%' || #{searchWrd} || '%' 		
    </if>	
    <if test="searchCnd == 2">AND
            b.USER_EMAIL LIKE '%' || #{searchWrd} || '%' 		
    </if>				

    <if test="bbsId == 'B0000000000000000003' or bbsId == 'B9999999999999999999' ">AND
        ( a.FRST_REGISTER_ID = #{ntcrId} 
            OR ( a.PARNTSCTT_NO IN ( SELECT NTT_ID FROM COMTNBBS WHERE BBS_ID = #{bbsId} AND USE_AT = 'Y' AND FRST_REGISTER_ID = #{ntcrId} ) )
        )
    </if>
            
    ORDER BY a.SORT_ORDR DESC, NTT_ID ASC
    ) TB ) WHERE rn BETWEEN #{firstIndex} + 1 AND #{firstIndex} + #{recordCountPerPage}
```

```
/* selectArticleListCnt */
SELECT
    COUNT(a.NTT_ID)
FROM
    COMTNBBS a
LEFT OUTER JOIN 
    TBL_OD_USER b ON a.frst_register_id = b.USER_SEQ
WHERE
    a.BBS_ID = #{bbsId}
AND a.USE_AT = 'Y'

<if test="searchCnd == 0">AND
        a.NTT_SJ LIKE '%' || #{searchWrd} || '%' 		
</if>
<if test="searchCnd == 1">AND
        a.NTT_CN LIKE '%' || #{searchWrd} || '%' 		
</if>	
<if test="searchCnd == 2">AND
        b.USER_EMAIL LIKE '%' || #{searchWrd} || '%' 		
</if>

<if test="bbsId == 'B0000000000000000003' or bbsId == 'B9999999999999999999' ">AND
    ( a.FRST_REGISTER_ID = #{ntcrId} 
        OR ( a.PARNTSCTT_NO IN ( SELECT NTT_ID FROM COMTNBBS WHERE BBS_ID = #{bbsId} AND USE_AT = 'Y' AND FRST_REGISTER_ID = #{ntcrId} ) )
    )
</if>
```

```
SELECT
    NTT_ID
    , BBS_ID
    , NTT_SJ
    , NTT_CN
FROM
(
    SELECT
        ROW_NUMBER() OVER (ORDER BY A.FRST_REGIST_PNTTM DESC) AS NUM
        , A.NTT_ID
        , A.BBS_ID
        , A.NTT_SJ
        , A.NTT_CN
    FROM COMTNBBS A
    WHERE 1=1
        AND A.BBS_ID = 'B0000000000000000001'
        AND A.NOTICE_AT = 'Y'
        AND A.USE_AT = 'Y'
        AND TO_CHAR(SYSDATE, 'YYYYMMDD') BETWEEN SUBSTR(A.NTCE_BGNDE,1,8) AND SUBSTR(A.NTCE_ENDDE,1,8)
    ORDER BY A.FRST_REGIST_PNTTM DESC
)
WHERE 1=1
AND NUM <= 3
```