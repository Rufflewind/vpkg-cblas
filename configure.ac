AC_INIT([cblas], [3.6.0])
AM_INIT_AUTOMAKE([foreign])
AC_PROG_CC()

dnl This is a prerequisite of AC_CHECK_HEADER.
dnl We need to explicitly invoke it here because
dnl our use of AC_CHECK_HEADER is conditional.
dnl
dnl AC_PROG_CPP

dnl Push variables onto their own stacks.
dnl
dnl     rf_push [-k] [<variable>...]
dnl
dnl By default, the variables are unset.  To prevent this, use the -k flag.
dnl
dnl The variable names must be valid and not contain any duplicates.
dnl
dnl --------------------------------------------------------------------------
dnl ensure prerequisites of AS_VAR_ARITH and AS_UNSET are expanded
AS_VAR_ARITH([_rf_scope_v], [0])
AS_UNSET(_rf_scope_v)
rf_push() {
    if test "[$]1" = -k; then
        _rf_scope_k=t
        shift
    else
        _rf_scope_k=f
    fi
    for _rf_scope_v
    do
        eval "_rf_scope_i=\${_rf_scope_i${_rf_scope_v}-0}"
        if eval "test x\${${_rf_scope_v}+x} = xx"; then
            eval "_rf_scope_s${_rf_scope_i}_${_rf_scope_v}=\${${_rf_scope_v}}"
            if test ${_rf_scope_k} = t; then
                AS_UNSET([${_rf_scope_v}])
            fi
        fi
        AS_VAR_ARITH([_rf_scope_i${_rf_scope_v}], [${_rf_scope_i} + 1])
    done
    AS_UNSET([_rf_scope_i])
    AS_UNSET([_rf_scope_k])
    AS_UNSET([_rf_scope_v])
}

dnl Pop variables off their own stacks.
dnl
dnl     rf_pop [-k] [<variable>...]
dnl
dnl The -k flag has no effect.
dnl
dnl The variable names must be valid and not contain any duplicates.
dnl
dnl --------------------------------------------------------------------------
dnl ensure prerequisites of AS_VAR_ARITH and AS_UNSET are expanded
AS_VAR_ARITH([_rf_scope_v], [0])
AS_UNSET(_rf_scope_v)
rf_pop() {
    if test "[$]1" = -k; then
        shift
    fi
    for _rf_scope_v
    do
        eval "_rf_scope_i=\${_rf_scope_i${_rf_scope_v}}"
        AS_VAR_ARITH([_rf_scope_i], [${_rf_scope_i} - 1])
        eval "_rf_scope_i${_rf_scope_v}=${_rf_scope_i}"
        if test ${_rf_scope_i} -eq 0; then
            AS_UNSET([_rf_scope_i${_rf_scope_v}])
        fi
        if eval "test x\${_rf_scope_s${_rf_scope_i}_${_rf_scope_v}+x} = xx"
        then
            eval "${_rf_scope_v}=\${_rf_scope_s${_rf_scope_i}_${_rf_scope_v}}"
            AS_UNSET([_rf_scope_s${_rf_scope_i}_${_rf_scope_v}])
        fi
    done
    AS_UNSET([_rf_scope_i])
    AS_UNSET([_rf_scope_v])
}

dnl Push the <variables>, run the given <action>, then pop the <variables>.
dnl
dnl     RF_SCOPE([<variables>], [<action>])
dnl
dnl The -k flag can be used in <variables>.
dnl
AC_DEFUN([RF_SCOPE], [{
    rf_push $1
    $2
    rf_pop $1
}])

dnl Check if the <function> can be linked using the flags in <libs>.
dnl
dnl     RF_CHECK_LIBS(<var>, <function>, [<libs>],
dnl                   [<if-found>], [<if-not-found>])
dnl
dnl If <var> is not empty, prepend the flags to the variable <var> upon
dnl success.
dnl
AC_DEFUN([RF_CHECK_LIBS], [{
    ifelse([$3], [],
        [AC_MSG_CHECKING([for $2])],
        [AC_MSG_CHECKING([for $2 in $3])]
    )
    rf_push -k LIBS
    LIBS="$3 ${LIBS-}"
    AC_LINK_IFELSE([AC_LANG_CALL([], [$2])], [
        rf_pop LIBS
        AC_MSG_RESULT(yes)
        $1="$3${$1:+ }${$1}"
        $4
    ], [
        rf_pop LIBS
        AC_MSG_RESULT(no)
        $5
    ])
}])

dnl Try to locate a header file <header> with the aid of the compiler.
dnl
dnl     RF_LOCATE_HEADER(<header>, [<if-found>], [<if-not-found>])
dnl
dnl The path is stored in the 'result' variable.  If the header file is not
dnl found, the result is empty.
dnl
AC_DEFUN([RF_LOCATE_HEADER], [{
    result=
    RF_SCOPE([output], [
        AS_IF([output=`printf "%s\n" "#include <$1>" |
                ${CC} -H -E -x c - -o /dev/null 2>&1`], [
            output=`printf "%s\n" "${output}" | sed -n 1p`
            AS_CASE([${output}],
            [". "*], [
                result=`printf "%s\n" "${output}" | sed 's/. //'`
            ])
        ])
    ])
    AS_IF([test x${result:+x} != x], [$2], [$3])
}])

AC_DEFUN([TEST_PROGRAM], [AC_LANG_PROGRAM([
    #undef NDEBUG
    #include <assert.h>
    #include <${header}>
], [
    double a = 1.0, b = 2.0, c = 3.0;
    cblas_dgemm(CblasRowMajor, CblasNoTrans, CblasNoTrans,
                1, 1, 1, 1.0, &a, 1, &b, 1, 1.0, &c, 1);
    assert(*c == 5.0);
])])

AC_DEFUN([CHECK_FLAGS], [{
    rf_push -k CFLAGS LIBS result
    CFLAGS="${CFLAGS-} ${CBLAS_CFLAGS-}"
    LIBS="${CBLAS_LIBS-} ${LIBS}"
    result=no
    AC_CHECK_HEADER([${header}], [result=yes])
    AS_UNSET([ac_cv_header_cblas_h])
    AS_IF([test ${result} != yes], [
        rf_pop CFLAGS LIBS result
        $2
    ], [
        AC_MSG_CHECKING([if CBLAS can be linked])
        AC_LINK_IFELSE([TEST_PROGRAM], [result=yes])
        AC_MSG_RESULT(${result})
        AS_IF([test ${result} != yes], [
            rf_pop CFLAGS LIBS result
            $2
        ], [
            AC_MSG_CHECKING([if CBLAS works])
            AC_RUN_IFELSE([TEST_PROGRAM], [result=yes])
            AC_MSG_RESULT(${result})
            AS_IF([test ${result} != yes], [
                rf_pop CFLAGS LIBS result
                $2
            ], [
                rf_pop CFLAGS LIBS result
                $1
            ])
        ])
    ])
}])

header=cblas.h

found=no

AS_IF([test x${CBLAS_CFLAGS+x}${CBLAS_LIBS+x} != x], [
    AC_MSG_NOTICE([note: CBLAS_CFLAGS and/or CBLAS_LIBS have been set])
    AC_MSG_NOTICE([      library auto-detection will not occur])
    CHECK_FLAGS([found=yes], [AC_MSG_ERROR([CBLAS library cannot be used])])
])

AS_IF([test ${found} = no], [
    PKG_CHECK_MODULES([CBLAS], [cblas], [
        CHECK_FLAGS([found=yes], [])
    ], [:])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [], [
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-lmkl_rt], [
        header=mkl_cblas.h
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-lopenblas], [
        AC_MSG_CHECKING([for OpenBLAS/cblas.h])
        RF_LOCATE_HEADER([OpenBLAS/cblas.h], [
            CBLAS_CFLAGS=-I`dirname "${result}"`
            AC_MSG_RESULT(${CBLAS_CFLAGS})
        ], [
            AC_MSG_RESULT(no)
        ])
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-framework Accelerate], [
        AC_MSG_CHECKING([for Accelerate framework headers])
        RF_LOCATE_HEADER([Accelerate/Accelerate.h], [
            result=`dirname "${result}"`
            result=`dirname "${result}"`
            result=${result}/Frameworks/vecLib.framework/Headers
            CBLAS_CFLAGS=-I${result}
            AC_MSG_RESULT(${CBLAS_CFLAGS})
        ], [
            AC_MSG_RESULT(no)
        ])
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-lcblas -latlas], [
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-lcblas], [
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    RF_CHECK_LIBS([CBLAS_LIBS], [cblas_dgemm], [-lblas], [
        CHECK_FLAGS([found=yes], [])
    ])
])

AS_IF([test ${found} = no], [
    AC_MSG_ERROR([could not find a usable CBLAS library])
])

if test ${header} != cblas.h; then
    CBLAS_CFLAGS=${CBLAS_CFLAGS}${CBLAS_CFLAGS:+ }-I${includedir}
    AC_CONFIG_FILES([cblas.h])
    headers=cblas.h
fi
AC_SUBST([header])
AC_SUBST([headers])
AC_CONFIG_FILES([cblas.pc Makefile])
AC_OUTPUT
