# TypeORM 패턴
TypeORM 에서는 **Active Record** 와 **Data Mapper** 패턴을 지원합니다.   
**Data Mapper 는 큰 서비스**에서 유지보수하며 개발하기 좋다는 장점이 있고,
**Active Record 는 간단하기에 작은 서비스**에서 유지보수하면서 사용하기 좋습니다.

## Active Record Pattern 
Model 을 테이블로 보기에 모델에서 바로 메서드를 사용할 수 있습니다.
Entity 에서 쿼리 메서드를 정의하고, 쿼리 메소드를 통해 객체를 조회, 저장, 삭제합니다.
또한 모든 Active Record entity 들은 BaseEntity 를 상속해야하며 이것은 여러가지 메서드들을 제공합니다(save, remove, find,...).
BaseEntity 는 기본적인 Repository 가 가진 메서드를 수행가능합니다.    
Active Record 를 사용한다면 Repository 나 Entity Manager 를 사용할 필요가 없습니다.

```typescript
import {
    Column,
    CreateDateColumn,
    PrimaryGeneratedColumn,
    UpdateDateColumn,
    Entity,
    BaseEntity,
} from 'typeorm';

@Entity()
export class Notice extends BaseEntity {
    @PrimaryGeneratedColumn()
    idx: number;
    
    @CreateDateColumn({
        type: 'timestamp',
    })
    createdAt: Date;
    
    @UpdateDateColumn({
        type: 'timestamp',
    })
    updatedAt: Date;
    
    @Column({ type: 'boolean', default: true })
    status: boolean;
    
    @Column({ length: 100 })
    title: string;

    @Column({ length: 1000 })
    contents: string;
  
    static findNotice(noticeIdx: number) {
      return this.findOne(noticeIdx);
    }
}

// Use(사용)
const findNoticeResult = await Notice.findNotice(1);
```

## Data Mapper Pattern
Model 에서 바로 사용하는 것이 아니라 Repository 에서 객체의 조회, 저장, 삭제 등을 수행합니다.
즉, repository 를 경유해서 DB 연산을 수행한다는 것입니다.   
그렇기에 DB와 Model 을 완전히 분리시키며 Mapper 가 모델과 DB 사이에서 데이터를 이동시키는 역할을 해줍니다.
Entity 클래스는 DB 에 대해서 의존성이 없다는 것을 의미합니다.   
이 때문에 큰 서비스일 수록 사용하기 좋다는 장점이 있습니다.

```typescript
// xxx.repository.ts
import {
  EntityRepository,
  Repository,
  TransactionManager,
  EntityManager,
} from 'typeorm';
import { Notice } from '../../entities/notice/notice.entity';
import { Injectable } from '@nestjs/common';

@Injectable()
@EntityRepository(Notice)
export class NoticeRepository extends Repository<Notice> {
  async registerProject(projectInfo): Promise<Project[]> {
    const project: Project[] = this.create(projectInfo);

    return this.save(project);
  }
}

// Use(사용)
// xxx.service.ts
export class NoticeService {
    constructor(
        @InjectRepository(NoticeRepository)
        private noticeRepository: NoticeRepository,
    ) {
        this.noticeRepository = noticeRepository;
    }
    //...
    async registerNotice (noticeInfo) {
        const registerNoticeResult = await this.projectRepository.registerProject(noticeInfo);    
    }
}
```

어느 것이 무조건 좋은 방식이라고 하기에는 어렵고, 상황에 따라서 적절하게 잘 사용하는 것이 좋다고 합니다.
