**\- 트랜잭션 처리가 중요한 이유**

만약 하나의 요청을 통해 여러개의 insert, update, delete문이 실행되어야 할 경우 트랜잭션은 아주 중요합니다.

![다운로드](https://user-images.githubusercontent.com/63203480/139619854-ddae7f5d-7da9-4559-ac7e-5f4e5b534a85.png)

만약 요청 시 insert, update, delete가 처리가 되는데, 위의 사진처럼 update문 다음에 서버에서 오류가 발생할 경우
**insert, update**문은 실행되지만 다음 **delete**문이 실행되지 못하는 경우가 발생합니다.

따라서 이럴 때 트랜잭션 처리를 해준다면 요청 중 오류가 발생할 경우 **rollback**을 하여 요청이 실행되기 전 상태로 되돌아가고, 만약 정상적으로 처리가 이루어질 경우 **commit**을 하여 서버에 변경 된 데이터를 반영할 수 있게 됩니다.

**\- Code(async/await)**

```
const {pool} = require("../../../config/database");
const connection = await pool.getConnection(async (conn) => conn);
try {
    const wishPersonInfo = [adultGuestNum, childGuestNum, infantGuestNum, userIdx, wishIdx];
    await connection.beginTransaction();
    const updateWishPersonResult = await wishListDao.updateWishPerson(connection, wishPersonInfo);//query 실행
    await connection.commit();

    return response(baseResponse.WISHLISTS_PERSON_UPDATE_SUCCESS, updateWishPersonResult[0].info);
} catch (err) {
    await connection.rollback();
    logger.error(`App - deleteWishList Service error\n: ${err.message}`);
    return errResponse(baseResponse.DB_ERROR);
} finally {
    connection.release();
}
```

- connection.beginTransaction() : 트랜잭션 적용 시작

- connection.commit() : 트랜잭션 반영

- connection.rollback() : 트랜잭션 롤백(이전 상태로 돌림)

- connection.release() : connection 해제